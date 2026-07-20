---
name: frontal
description: Foundational skill for building with the Frontal TypeScript SDK ecosystem (@frontal-labs/*). Covers the unified Sdk class, core client patterns, configuration, error handling, retries, pagination, and environment setup. Use whenever working with any Frontal SDK package — this is the primary entry point for Frontal SDK usage. Triggers on "Frontal SDK", "@frontal-labs", "FrontalClient", "Sdk class", "frontal API", or any domain-specific Frontal SDK task.
metadata:
  internal: true
  author: frontal-labs
  version: "1.0"
  category: sdk
---

# Frontal SDK: Core Patterns

## When to Use

- Importing or configuring any `@frontal-labs/*` package
- Creating SDK clients (standalone or composed)
- Handling Frontal API errors, retries, or pagination
- Setting up environment variables for SDK usage
- Choosing between the unified `Sdk` class vs. individual service clients
- Building applications that call `api.frontal.dev/v1/*` or `ai.frontal.dev`

## Quick Start

```bash
bun add @frontal-labs/sdk
```

```typescript
import { sdk } from "@frontal-labs/sdk";

const result = await sdk.ai.generateText({
  model: "gpt-4o-mini",
  prompt: "Hello, Frontal!"
});
```

The `sdk` singleton reads `FRONTAL_API_KEY` from the environment automatically.

## Unified SDK vs. Individual Clients

### Option A: Unified `Sdk` class (recommended for most apps)

```typescript
import { createSdkClient, sdk } from "@frontal-labs/sdk";

const sdk = createSdkClient({
  apiKey: process.env.FRONTAL_API_KEY!,
  baseUrl: "https://api.frontal.dev/v1"
});

// Lazy-loaded accessors — only the services you use are initialized
await sdk.ai.generateText({ ... });
await sdk.billing.subscriptions.list();
await sdk.graph.entities("customer").list();
```

The `Sdk` class aggregates **all 20+ services** as lazy-loaded getters and re-exports
every individual service singleton for tree-shaking.

### Option B: Individual service client (recommended for libraries/workers)

```typescript
import { createAIClient } from "@frontal-labs/ai";
import { createBillingClient } from "@frontal-labs/billing";

const ai = createAIClient({ apiKey, baseUrl: "https://ai.frontal.dev" });
const billing = createBillingClient({ apiKey, baseUrl: "https://api.frontal.dev/v1" });
```

Use individual clients when you need a **different base URL or auth per service**, or
when you want to avoid importing the full SDK bundle.

### Option C: Composed via `FrontalClient` (recommended for shared HTTP config)

```typescript
import { FrontalClient } from "@frontal-labs/core";
import { AIService } from "@frontal-labs/ai";

const core = new FrontalClient({
  apiKey,
  baseUrl: "https://api.frontal.dev/v1",
  timeout: 30_000,
  maxRetries: 2
});

const ai = new AIService(core.httpClient);
```

All `create<Package>Client` factories accept either a `FrontalClient` instance or a
plain config object. Passing a `FrontalClient` shares the HTTP connection pool and
retry configuration across services.

## Client Creation Patterns (Pattern B)

Every package follows the same export pattern:

| Export | Type | Purpose |
|--------|------|---------|
| `create<Package>Client(config)` | Factory function | Create from config object or `FrontalClient` |
| `<package>` | `Proxy` singleton | Lazy-initialized default instance (reads env vars) |
| `<Package>Service` | Class | Direct instantiation for advanced usage |
| `VERSION` | `string` | Package version |
| `schemas` | Zod schemas + types | Runtime validation + TypeScript types |

```typescript
// Factory
const ai = createAIClient({ apiKey, baseUrl });

// Singleton (lazy, env-based)
const result = await ai.generateText({ ... });  // reads FRONTAL_API_KEY automatically

// Direct class
const service = new AIService(httpClient);
```

## Environment Variables

| Variable | Default | Purpose |
|----------|---------|---------|
| `FRONTAL_API_KEY` | — | Global API key (required) |
| `FRONTAL_API_URL` | `https://api.frontal.dev/v1` | Default base URL for most services |
| `FRONTAL_AI_API_URL` | `https://ai.frontal.dev` | AI inference gateway |
| `FRONTAL_AGENTS_API_URL` | Falls back to `FRONTAL_API_URL` | Agent runtime |
| `FRONTAL_BILLING_API_URL` | Falls back to `FRONTAL_API_URL` | Billing service |
| `FRONTAL_WORKFLOWS_API_URL` | Falls back to `FRONTAL_API_URL` | Workflow service |
| `FRONTAL_WORKERS_API_URL` | Falls back to `FRONTAL_API_URL` | Edge runtime |
| `FRONTAL_SANDBOX_API_URL` | Falls back to `FRONTAL_API_URL` | Sandbox service |

```bash
# Minimal setup
export FRONTAL_API_KEY="frt_..."
export FRONTAL_API_URL="https://api.frontal.dev/v1"

# Per-service override
export FRONTAL_AI_API_URL="https://ai.frontal.dev"
```

## Configuration Object

```typescript
interface ClientConfigInput {
  apiKey: string;
  baseUrl?: string;
  timeout?: number;        // ms, default 30_000
  maxRetries?: number;     // default 3
  retryDelay?: number;     // ms, default 1_000
  headers?: Record<string, string>;
  environment?: "production" | "staging" | "development";
  debug?: boolean;
}
```

## Error Handling

All SDK errors extend `FrontalError` and carry a `name`, `message`, and `context`.
The core package exports typed error classes:

| Error Class | HTTP Status | When |
|-------------|-------------|------|
| `FrontalError` | — | Base class |
| `NotFoundError` | 404 | Resource does not exist |
| `UnauthorizedError` | 401 | Missing or invalid API key |
| `ForbiddenError` | 403 | Insufficient permissions |
| `ValidationError` | 422 | Request body/params failed validation |
| `ConflictError` | 409 | Resource conflict (e.g., duplicate) |
| `RateLimitError` | 429 | Too many requests |
| `ServiceError` | 500-599 | Upstream service failure |
| `NetworkError` | — | Connection failed |
| `TimeoutError` | — | Request timed out |

```typescript
import {
  FrontalError,
  NotFoundError,
  RateLimitError,
  parseFrontalError
} from "@frontal-labs/core";

try {
  await sdk.billing.subscriptions.list();
} catch (err) {
  if (err instanceof NotFoundError) {
    console.log("No subscriptions found");
  } else if (err instanceof RateLimitError) {
    console.log(`Retry after ${err.retryAfter}ms`);
  } else if (err instanceof FrontalError) {
    console.error(`Frontal error [${err.name}]: ${err.message}`);
  }
}
```

## Retries, Circuit Breaker, and Timeouts

### Automatic Retries

The `HttpClient` retries on transient failures with exponential backoff + jitter:

```typescript
const client = new FrontalClient({
  apiKey,
  maxRetries: 3,
  retryDelay: 1000,
  retryOn: [NetworkError, TimeoutError, ServiceError]
});
```

### Circuit Breaker

Prevent cascading failures when a downstream service is unhealthy:

```typescript
import { CircuitBreaker, CircuitBreakerConfig } from "@frontal-labs/core";

const breaker = new CircuitBreaker({
  failureThreshold: 5,
  resetTimeout: 30_000,
  halfOpenMaxCalls: 3
} as CircuitBreakerConfig);

try {
  const result = await breaker.execute(() => sdk.ai.generateText({ ... }));
} catch (err) {
  if (err instanceof CircuitBreakerOpenError) {
    console.log("Circuit open — service is degraded");
  }
}
```

### Polling Utilities

For long-running operations, use `pollUntil`:

```typescript
import { pollUntil } from "@frontal-labs/core";

const run = await sdk.workflows.define("my-wf").create();
const result = await pollUntil(
  () => sdk.workflows.use(run.id).get(),
  {
    interval: 2_000,
    timeout: 120_000,
    until: (r) => r.status === "completed" || r.status === "failed"
  }
);
```

## Pagination

All list endpoints return `PageResult<T>` with cursor-based or offset-based pagination:

```typescript
const page = await sdk.billing.subscriptions.list({ limit: 20 });

// Async iteration
for await (const subscription of page) {
  console.log(subscription.id);
}

// Manual pagination
const all = await page.all();

// Or use nextPage()
const next = await page.nextPage();
```

## Typed Responses

Every API response is wrapped in `APIResponse<T>`:

```typescript
interface APIResponse<T> {
  data: T;
  error?: ErrorResponse;
  meta: ResponseMeta;       // requestId, timestamp, version
}
```

Use the `ResponseMeta.requestId` for support investigations.

## Best Practices

- **Prefer the unified `Sdk` class** for applications; use individual clients for libraries
- **Share `FrontalClient`** when composing multiple services in the same process
- **Always catch typed errors** (`instanceof FrontalError`) rather than generic `Error`
- **Use `pollUntil`** for workflow/agent run completion instead of raw `setInterval`
- **Set `debug: true`** in development to log request/response payloads
- **Use environment-specific `baseUrl`** — never hardcode production URLs

## Common Pitfalls

- **Do not** import from internal paths like `@frontal-labs/core/src/...` — use public exports
- **Do not** mix `sdk.ai` (singleton) with `createAIClient()` pointing to a different base URL — they are separate instances
- **Do not** ignore `RateLimitError` — always back off using `retryAfter` if present
- **Do not** call list endpoints in a loop without pagination — use `all()` or async iteration

## References

- `references/sdk-architecture.md` — Deep dive into package structure, build system, and dependency graph
- `references/error-handling.md` — Comprehensive error taxonomy and recovery strategies
- `references/pagination-patterns.md` — Cursor vs. offset pagination, async iteration, backpressure
- `references/testing-patterns.md` — Using `@frontal-labs/testing` for unit and integration tests
