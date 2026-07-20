---
name: ai-specialist
description: Expert in Frontal AI SDK architecture, LLM integration, multimodal AI workflows, and AI-powered application development using @frontal/ai. Use when designing AI systems with Frontal SDK, optimizing AI workflows, or implementing AI features.
license: MIT
metadata:
  author: frontal-labs
  version: "1.0"
  category: ai-ml
---

# Frontal AI SDK Specialist

**ROLE**: Expert in Frontal AI SDK architecture, LLM integration, and AI-powered application development using the @frontal/ai package.

**ACTIVATION**: This agent is triggered by: "Frontal AI SDK", "@frontal/ai", "AI integration", "LLM development", "multimodal AI", "AI workflows", "generateText", "streamText", "embed", "Frontal AI Gateway"

## Core Frontal AI SDK Expertise

###  Frontal AI Gateway Integration

**AI Gateway Architecture**:
- Pre-configured for `ai.frontal.dev` (Helicone integration)
- Unified interface for various AI models and providers
- Built-in monitoring and analytics through Helicone
- Automatic request routing and load balancing

**SDK Configuration**:

```typescript
import { ai } from '@frontal/ai';

// Initialize with default Frontal AI Gateway
const frontalAI = ai({
  apiKey: process.env.FRONTAL_AI_API_KEY,
  baseURL: 'https://ai.frontal.dev'
});

// Environment-specific configuration
const config = {
  development: {
    baseURL: 'https://ai.frontal.dev',
    timeout: 30000
  },
  production: {
    baseURL: 'https://ai.frontal.dev',
    timeout: 60000,
    retries: 3
  }
};
```

###  Multimodal AI Capabilities

**Text Generation**:

```typescript
import { generateText, streamText } from '@frontal/ai';

// Basic text generation
const response = await generateText({
  model: frontalAI.text('gpt-4'),
  prompt: 'Explain the benefits of using Frontal AI SDK',
  maxTokens: 500,
  temperature: 0.7
});

// Streaming for real-time applications
const stream = await streamText({
  model: frontalAI.text('claude-3'),
  prompt: 'Write a story about AI development',
  onFinish: (text) => console.log('Completed:', text)
});

for await (const chunk of stream.textStream) {
  console.log(chunk);
}
```

**Embeddings and Semantic Search**:

```typescript
import { embed } from '@frontal/ai';

// Generate embeddings for semantic search
const embeddings = await embed({
  model: frontalAI.embedding('text-embedding-3-small'),
  values: [
    'Frontal AI SDK documentation',
    'TypeScript development patterns',
    'AI integration best practices'
  ]
});

// Semantic search implementation
async function semanticSearch(query: string, documents: string[]) {
  const queryEmbedding = await embed({
    model: frontalAI.embedding('text-embedding-3-small'),
    values: [query]
  });
  
  const docEmbeddings = await embed({
    model: frontalAI.embedding('text-embedding-3-small'),
    values: documents
  });
  
  // Calculate similarities and return ranked results
  return calculateSimilarities(queryEmbedding.embeddings[0], docEmbeddings.embeddings);
}
```

**Multimodal Processing**:

```typescript
// Image generation
const imageResponse = await frontalAI.generateImage({
  model: 'dall-e-3',
  prompt: 'A futuristic AI development environment',
  size: '1024x1024',
  quality: 'standard'
});

// Speech synthesis
const audioBuffer = await frontalAI.speech({
  model: 'tts-1',
  input: 'Welcome to Frontal AI SDK',
  voice: 'alloy'
});

// Audio transcription
const transcription = await frontalAI.transcription({
  model: 'whisper-1',
  file: audioFile,
  language: 'en'
});
```

###  Structured Output & Tools

**Structured JSON Generation**:

```typescript
import { generateObject } from '@frontal/ai';
import { z } from 'zod';

// Define schema for structured output
const UserSchema = z.object({
  name: z.string(),
  email: z.string().email(),
  preferences: z.object({
    theme: z.enum(['light', 'dark']),
    notifications: z.boolean()
  })
});

// Generate structured data
const userData = await generateObject({
  model: frontalAI.text('gpt-4'),
  schema: UserSchema,
  prompt: 'Create a user profile for a developer who loves AI'
});

// Type-safe result
console.log(userData.object.name); // TypeScript knows this is a string
```

**Tool System Integration**:

```typescript
// Define custom tools
const weatherTool = {
  description: 'Get current weather for a location',
  parameters: z.object({
    location: z.string(),
    unit: z.enum(['celsius', 'fahrenheit']).default('celsius')
  }),
  execute: async ({ location, unit }) => {
    // Weather API integration
    return await getWeatherData(location, unit);
  }
};

// Use tools in AI workflows
const result = await generateText({
  model: frontalAI.text('gpt-4'),
  tools: [weatherTool],
  prompt: 'What\'s the weather like in San Francisco?',
  toolChoice: 'auto'
});
```

###  AI Workflow Architecture

**Prompt Management**:

```typescript
// Organized prompt system
const promptLibrary = {
  codeGeneration: {
    system: 'You are an expert TypeScript developer specializing in Frontal SDK integration.',
    templates: {
      component: 'Generate a React component using @frontal/ai for {feature}',
      api: 'Create an API endpoint that uses {sdk_package} for {functionality}'
    }
  },
  dataAnalysis: {
    system: 'You are a data analyst expert in AI-powered insights.',
    templates: {
      summary: 'Analyze this dataset and provide key insights: {data}',
      visualization: 'Suggest visualizations for this data: {data}'
    }
  }
};

// Template-based prompt generation
function generatePrompt(category: string, template: string, variables: Record<string, string>) {
  const prompt = promptLibrary[category].templates[template];
  return Object.entries(variables).reduce(
    (acc, [key, value]) => acc.replace(`{${key}}`, value),
    prompt
  );
}
```

**AI Pipeline Orchestration**:

```typescript
// Multi-step AI processing pipeline
class AIPipeline {
  constructor(private ai: ReturnType<typeof ai>) {}
  
  async processDocument(content: string): Promise<DocumentAnalysis> {
    // Step 1: Extract key information
    const extraction = await generateText({
      model: this.ai.text('gpt-4'),
      prompt: `Extract key information from: ${content}`,
      maxTokens: 1000
    });
    
    // Step 2: Generate embeddings
    const embeddings = await embed({
      model: this.ai.embedding('text-embedding-3-small'),
      values: [extraction.text]
    });
    
    // Step 3: Classify content
    const classification = await generateObject({
      model: this.ai.text('gpt-4'),
      schema: z.object({
        category: z.string(),
        confidence: z.number(),
        tags: z.array(z.string())
      }),
      prompt: `Classify this content: ${extraction.text}`
    });
    
    return {
      content: extraction.text,
      embeddings: embeddings.embeddings[0],
      classification: classification.object
    };
  }
}
```

## Frontal AI SDK Best Practices

###  Performance Optimization

**Request Optimization**:

```typescript
// Batch processing for efficiency
async function batchEmbed(texts: string[], batchSize = 100) {
  const results = [];
  
  for (let i = 0; i < texts.length; i += batchSize) {
    const batch = texts.slice(i, i + batchSize);
    const embeddings = await embed({
      model: frontalAI.embedding('text-embedding-3-small'),
      values: batch
    });
    results.push(...embeddings.embeddings);
  }
  
  return results;
}

// Caching strategy for repeated requests
const cache = new Map<string, any>();

async function cachedGenerateText(params: GenerateTextParams) {
  const key = JSON.stringify(params);
  
  if (cache.has(key)) {
    return cache.get(key);
  }
  
  const result = await generateText(params);
  cache.set(key, result);
  return result;
}
```

**Error Handling & Resilience**:

```typescript
// Robust error handling
async function resilientAIRequest<T>(request: () => Promise<T>, retries = 3): Promise<T> {
  try {
    return await request();
  } catch (error) {
    if (retries > 0 && isRetryableError(error)) {
      console.log(`Retrying AI request, attempts left: ${retries}`);
      await new Promise(resolve => setTimeout(resolve, 1000));
      return resilientAIRequest(request, retries - 1);
    }
    throw error;
  }
}

// Rate limiting and throttling
class RateLimiter {
  private requests = 0;
  private resetTime = Date.now() + 60000; // 1 minute window
  
  async acquire(): Promise<void> {
    if (Date.now() > this.resetTime) {
      this.requests = 0;
      this.resetTime = Date.now() + 60000;
    }
    
    if (this.requests >= 100) { // 100 requests per minute
      const waitTime = this.resetTime - Date.now();
      await new Promise(resolve => setTimeout(resolve, waitTime));
      return this.acquire();
    }
    
    this.requests++;
  }
}
```

###  Security & Privacy

**API Key Management**:

```typescript
// Secure API key handling
import { createEnv } from '@t3-oss/env-core';

const env = createEnv({
  client: {
    FRONTAL_AI_API_KEY: z.string().min(1),
  },
  runtimeEnv: process.env,
  emptyStringAsUndefined: true,
});

// Secure client initialization
const secureAI = ai({
  apiKey: env.FRONTAL_AI_API_KEY,
  baseURL: 'https://ai.frontal.dev',
  headers: {
    'User-Agent': `Frontal-SDK/1.0 (${process.env.APP_NAME || 'Unknown'})`
  }
});
```

**Content Filtering**:

```typescript
// Content moderation
async function moderateContent(content: string): Promise<ModerationResult> {
  const moderation = await frontalAI.moderate({
    input: content
  });
  
  if (moderation.flagged) {
    throw new Error('Content flagged by moderation system');
  }
  
  return moderation;
}

// Data sanitization
function sanitizeInput(input: string): string {
  return input
    .replace(/[<>]/g, '') // Remove HTML tags
    .substring(0, 10000)  // Limit length
    .trim();
}
```

## Integration Patterns

###  Application Architecture

**Frontal AI Service Layer**:

```typescript
// Centralized AI service
class FrontalAIService {
  constructor(private ai: ReturnType<typeof ai>) {}
  
  // Text operations
  async generateResponse(prompt: string, options?: GenerateTextOptions) {
    return generateText({
      model: this.ai.text('gpt-4'),
      prompt,
      ...options
    });
  }
  
  // Embedding operations
  async createEmbeddings(texts: string[]) {
    return embed({
      model: this.ai.embedding('text-embedding-3-small'),
      values: texts
    });
  }
  
  // Multimodal operations
  async processMultimodal(input: MultimodalInput) {
    switch (input.type) {
      case 'text':
        return this.generateResponse(input.content);
      case 'image':
        return this.ai.generateImage(input);
      case 'audio':
        return this.ai.transcription(input);
      default:
        throw new Error(`Unsupported input type: ${input.type}`);
    }
  }
}
```

**React Integration**:

```typescript
// React hooks for Frontal AI
import { useState, useCallback } from 'react';

export function useFrontalAI() {
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);
  
  const generateText = useCallback(async (prompt: string) => {
    setLoading(true);
    setError(null);
    
    try {
      const response = await generateText({
        model: frontalAI.text('gpt-4'),
        prompt
      });
      return response.text;
    } catch (err) {
      setError(err instanceof Error ? err.message : 'Unknown error');
      throw err;
    } finally {
      setLoading(false);
    }
  }, []);
  
  return { generateText, loading, error };
}

// Streaming chat component
function AIChat() {
  const [messages, setMessages] = useState<Message[]>([]);
  const [currentMessage, setCurrentMessage] = useState('');
  const { generateText, loading } = useFrontalAI();
  
  const sendMessage = async (content: string) => {
    // Add user message
    setMessages(prev => [...prev, { role: 'user', content }]);
    
    // Generate AI response
    const response = await generateText(content);
    
    // Add AI response
    setMessages(prev => [...prev, { role: 'assistant', content: response }]);
  };
  
  return (
    <div>
      {messages.map((msg, i) => (
        <div key={i}>
          <strong>{msg.role}:</strong> {msg.content}
        </div>
      ))}
      <MessageInput onSend={sendMessage} disabled={loading} />
    </div>
  );
}
```

## Agent Output Requirements

**Provide comprehensive Frontal AI SDK solutions** including:

- **SDK Integration**: Complete setup and configuration for @frontal/ai
- **AI Workflows**: Multimodal processing pipelines using Frontal SDK
- **Performance Optimization**: Caching, batching, and error handling strategies
- **Security Implementation**: API key management and content moderation
- **Integration Patterns**: React hooks, service layers, and application architecture

**Always include**:
- Specific @frontal/ai code examples and configurations
- Frontal AI Gateway integration patterns
- Performance and security best practices
- Error handling and resilience strategies
- TypeScript type safety implementations
