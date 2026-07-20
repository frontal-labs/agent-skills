---
name: frontal-workers
description: Deploy and invoke serverless edge workers with @frontal-labs/workers — edge runtime, eszip bundles, environment variables, and request proxying. Use when deploying edge functions or serverless workloads on the Frontal edge runtime.
metadata:
  internal: true
  author: frontal-labs
  version: "1.0"
  category: sdk
---

# Frontal SDK: Workers

## When to Use

- Deploying serverless functions to the Frontal edge runtime
- Invoking workers by request path
- Managing worker environment variables and versions
- Any task mentioning `@frontal-labs/workers`, edge runtime, or serverless workers

## Quick Start

```bash
bun add @frontal-labs/workers
```

```typescript
import { workers } from "@frontal-labs/workers";

await workers.deploy({
  name: "image-resizer",
  code: `
    export default {
      fetch(request) {
        return new Response("Worker running");
      }
    }
  `,
  entrypoint: "index.ts"
});
```

## Client Setup

```typescript
import { createWorkersClient } from "@frontal-labs/workers";

const workers = createWorkersClient({
  apiKey: process.env.FRONTAL_API_KEY!,
  baseUrl: process.env.FRONTAL_WORKERS_API_URL ?? "https://api.frontal.dev/v1"
});
```

## Deploy

```typescript
// Deploy from source code string
await workers.deploy({
  name: "image-resizer",
  code: `export default { fetch(request) { ... } }`,
  entrypoint: "index.ts",
  envVars: {
    UPSTREAM_URL: "https://images.example.com"
  }
});

// Deploy from eszip bundle (binary)
await workers.deployBundle({
  name: "image-resizer",
  bundle: eszipBuffer,
  envVars: { UPSTREAM_URL: "https://images.example.com" }
});
```

## Invoke

```typescript
// Invoke by worker name and path
const response = await workers.invoke("image-resizer", "/resize?url=...");
// response.body, response.status, response.headers
```

## List and Manage

```typescript
// List workers
const list = await workers.list();

// Get worker details
const worker = await workers.get("image-resizer");

// Delete worker
await workers.delete("image-resizer");
```

## Best Practices

- **Keep workers under 1MB** — larger bundles cold-start slower
- **Use `envVars`** for configuration, not hardcoded URLs
- **Deploy with eszip** for production — it includes dependencies and is immutable
- **Set explicit entrypoint** — default is `index.ts`

## Common Pitfalls

- **Do not** deploy workers that depend on Node.js built-ins not supported by the edge runtime
- **Do not** store secrets in `code` — use `envVars` or secret references
- **Do not** invoke workers synchronously in high-throughput paths — use queue + async invocation

## References

- `references/workers-runtime.md` — Supported APIs, limitations, and polyfills
- `references/workers-bundling.md` — eszip format, bundling tools, and size optimization
