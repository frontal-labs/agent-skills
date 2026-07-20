---
name: frontal-ai
description: Expert skill for building AI features with @frontal-labs/ai — text generation, streaming, embeddings, structured outputs, multimodal (image/video/speech/transcription), and tool use. Use when the user mentions AI, LLMs, embeddings, generateText, streamText, embed, generateObject, or any @frontal-labs/ai integration.
metadata:
  internal: true
  author: frontal-labs
  version: "1.0"
  category: sdk
---

# Frontal SDK: AI & Inference

## When to Use

- Integrating LLM text generation or streaming into an application
- Generating embeddings for semantic search or RAG
- Producing structured JSON output with `generateObject`
- Building multimodal features: image generation, speech synthesis, transcription, video
- Implementing tool-calling / function-calling loops
- Managing prompt templates or moderation pipelines
- Any task mentioning `@frontal-labs/ai`, `ai.frontal.dev`, or `generateText`

## Quick Start

```bash
bun add @frontal-labs/ai
```

```typescript
import { ai, generateText, embed } from "@frontal-labs/ai";

const text = await generateText({
  model: "gpt-4o-mini",
  prompt: "Summarize this ticket in 3 bullets."
});

const vector = await embed({
  model: "text-embedding-3-small",
  input: "How do I reset my password?"
});
```

## Client Setup

```typescript
// Singleton (reads FRONTAL_API_KEY + FRONTAL_AI_API_URL automatically)
import { ai } from "@frontal-labs/ai";
const result = await ai.generateText({ model: "gpt-4o-mini", prompt: "Hi" });

// Factory (explicit config)
import { createAIClient } from "@frontal-labs/ai";
const client = createAIClient({
  apiKey: process.env.FRONTAL_API_KEY!,
  baseUrl: process.env.FRONTAL_AI_API_URL ?? "https://ai.frontal.dev"
});

// Composed with FrontalClient
import { FrontalClient } from "@frontal-labs/core";
import { AIService } from "@frontal-labs/ai";
const core = new FrontalClient({ apiKey, baseUrl: "https://api.frontal.dev/v1" });
const ai = new AIService(core.httpClient);
```

**Base URL defaults:**
- Singleton: `FRONTAL_AI_API_URL` or `https://ai.frontal.dev`
- Factory with config: same as above
- Composed with `FrontalClient`: inherits `baseUrl` from the core client

## Core APIs

### Text Generation

```typescript
const result = await generateText({
  model: "gpt-4o-mini",
  prompt: "Explain quantum computing",
  maxTokens: 500,
  temperature: 0.7,
  stop: ["\n\n\n"],
  headers: { "X-Session-Id": "abc" }
});
// result.text — generated string
// result.usage — { promptTokens, completionTokens, totalTokens }
// result.finishReason — "stop" | "length" | "content-filter"
```

### Streaming Text

```typescript
const stream = await streamText({
  model: "gpt-4o-mini",
  prompt: "Write a 500-word essay on AI",
  onChunk: (chunk) => process.stdout.write(chunk)
});

for await (const chunk of stream.textStream) {
  console.log(chunk);
}
// stream.final — complete response after streaming ends
```

### Embeddings

```typescript
const result = await embed({
  model: "text-embedding-3-small",
  input: [
    "Frontal is an enterprise domain-AI platform",
    "Agents orchestrate workflows across services"
  ]
});
// result.embeddings — number[][] (one vector per input)
// result.model — "text-embedding-3-small"
// result.usage — token usage
```

### Structured Output (generateObject)

```typescript
import { z } from "zod";

const result = await generateObject({
  model: "gpt-4o-mini",
  prompt: "Extract name, email, and plan from: 'Alice (alice@acme.com) is on Pro'",
  schema: z.object({
    name: z.string(),
    email: z.string().email(),
    plan: z.enum(["free", "pro", "enterprise"])
  })
});
// result.object — fully typed parsed object
// result.raw — unparsed model response
```

### Image Generation

```typescript
const image = await generateImage({
  model: "dall-e-3",
  prompt: "A futuristic AI development environment",
  size: "1024x1024",
  quality: "standard",
  style: "vivid"
});
// image.url — public URL
// image.revisedPrompt — what the model actually rendered
```

### Speech & Transcription

```typescript
// Text-to-speech
const audio = await generateSpeech({
  model: "tts-1",
  input: "Welcome to Frontal AI SDK",
  voice: "alloy"
});
// audio.buffer — AudioBuffer
// audio.contentType — "audio/mpeg"

// Audio transcription
const transcript = await transcription({
  model: "whisper-1",
  file: audioBlob,
  language: "en"
});
// transcript.text — transcribed string
// transcript.duration — seconds
```

### Moderation

```typescript
const mod = await moderation({
  input: "User-generated content here",
  model: "omni-moderation-latest"
});
// mod.flagged — boolean
// mod.categories — { sexual, hate, self-harm, ... }
// mod.categoryScores — { sexual: 0.01, hate: 0.00, ... }
```

### Tool Calling

```typescript
const result = await generateText({
  model: "gpt-4o-mini",
  prompt: "What is the weather in Paris?",
  tools: {
    getWeather: {
      description: "Get current weather for a city",
      parameters: z.object({
        city: z.string(),
        unit: z.enum(["c", "f"]).default("c")
      }),
      execute: async ({ city, unit }) => {
        const data = await fetch(`https://wttr.in/${city}?format=3`);
        return data.text();
      }
    }
  },
  maxSteps: 5,
  toolChoice: "auto"
});
// result.text — final response after tool calls
// result.toolCalls — array of tool invocations made
```

### Model Listing

```typescript
const models = await ai.models.list();
// models — array of { id, ownedBy, created, ... }
```

## Error Handling

AI-specific errors:
- `RateLimitError` — back off using `retryAfter` (ms)
- `ServiceError` — model overloaded or gateway error (retry with backoff)
- `ValidationError` — invalid schema in `generateObject`
- `ForbiddenError` — model access not permitted for your tier

```typescript
try {
  await generateText({ model: "gpt-4o", prompt: "..." });
} catch (err) {
  if (err instanceof RateLimitError) {
    await new Promise(r => setTimeout(r, err.retryAfter ?? 2000));
    return generateText({ ... }); // retry
  }
  throw err;
}
```

## Best Practices

- **Use `gpt-4o-mini` for most tasks** — cheaper and faster than `gpt-4o`
- **Use `streamText` for user-facing output** — perceived latency is much lower
- **Always set `maxTokens`** for deterministic response sizing
- **Use `generateObject` with Zod schemas** instead of parsing raw text
- **Cache embeddings** — they are deterministic for the same model+input
- **Implement exponential backoff** on `RateLimitError` and `ServiceError`

## Common Pitfalls

- **Do not** pass unsupported parameters to `generateText` — the gateway rejects unknown fields
- **Do not** stream inside a serverless function without consuming the stream to completion
- **Do not** embed large batches (>100 texts) in a single call — split into chunks
- **Do not** rely on `stop` sequences for structured output — use `generateObject`

## References

- `references/ai-models.md` — Supported models, capabilities, pricing tiers
- `references/ai-streaming.md` — Streaming patterns, backpressure, cancellation
- `references/ai-tools.md` — Tool definition schema, parallel tool calls, maxSteps
- `references/ai-moderation.md` — Content filtering policies and thresholds
