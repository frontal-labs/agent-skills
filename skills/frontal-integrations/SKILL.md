---
name: frontal-integrations
description: Execute actions in third-party applications with @frontal-labs/integrations — CRM sync, messaging, and external service orchestration. Use when building cross-service workflows or data synchronization with external tools.
metadata:
  internal: true
  author: frontal-labs
  version: "1.0"
  category: sdk
---

# Frontal SDK: Integrations

## When to Use

- Syncing data between Frontal and third-party apps (Salesforce, Slack, Linear, etc.)
- Triggering actions in external services from Frontal workflows
- Managing OAuth connections and integration configurations
- Any task mentioning `@frontal-labs/integrations` or third-party actions

## Quick Start

```bash
bun add @frontal-labs/integrations
```

```typescript
import { integrations } from "@frontal-labs/integrations";

await integrations.actions.execute("salesforce", "create_contact", {
  firstName: "Alice",
  lastName: "Smith",
  email: "alice@acme.com"
});
```

## Client Setup

```typescript
import { createIntegrationsClient } from "@frontal-labs/integrations";

const integrations = createIntegrationsClient({
  apiKey: process.env.FRONTAL_API_KEY!,
  baseUrl: "https://api.frontal.dev/v1"
});
```

## Connections

```typescript
// List available integrations
const available = await integrations.catalog.list();
// Salesforce, Slack, Linear, Notion, Zendesk, Intercom, etc.

// Create connection (OAuth flow)
const connection = await integrations.connections.create({
  provider: "salesforce",
  config: { instanceUrl: "https://acme.my.salesforce.com" }
});

// List active connections
const connections = await integrations.connections.list();

// Test connection
const healthy = await integrations.connections.test("conn_abc");
```

## Actions

```typescript
// Execute action
const result = await integrations.actions.execute("salesforce", "create_contact", {
  firstName: "Alice",
  lastName: "Smith",
  email: "alice@acme.com"
});

// List available actions for a provider
const actions = await integrations.actions.list("slack");
// post_message, upload_file, create_channel, etc.

// Execute with timeout and retry
const result = await integrations.actions.execute("linear", "create_issue", {
  team: "ENG",
  title: "Fix login bug",
  priority: "high"
}, {
  timeout: 10000,
  retries: 2
});
```

## Best Practices

- **Use short timeouts** (5-10s) for synchronous external calls
- **Implement async wrappers** for long-running integrations (upload, sync)
- **Cache connection tokens** — the SDK handles refresh automatically
- **Log action inputs/outputs** for debugging cross-service issues

## Common Pitfalls

- **Do not** execute integrations synchronously in user-facing request paths
- **Do not** retry non-idempotent actions (create, send) without checking for duplicates
- **Do not** store OAuth tokens manually — use `connections.create()` which handles secure storage

## References

- `references/integrations-providers.md` — Provider-specific setup and action catalogs
- `references/integrations-errors.md` — Provider error codes and retry strategies
