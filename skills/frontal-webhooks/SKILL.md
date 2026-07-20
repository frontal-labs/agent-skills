---
name: frontal-webhooks
description: Manage webhook endpoints with @frontal-labs/webhooks — create endpoints, verify HMAC signatures, track delivery, and configure retry rules. Use when receiving or delivering webhooks with Frontal.
metadata:
  internal: true
  author: frontal-labs
  version: "1.0"
  category: sdk
---

# Frontal SDK: Webhooks

## When to Use

- Creating webhook endpoints to receive Frontal events
- Verifying HMAC signatures for security
- Tracking delivery status and retry history
- Configuring retry policies and timeouts
- Any task mentioning `@frontal-labs/webhooks`, webhook endpoints, or HMAC verification

## Quick Start

```bash
bun add @frontal-labs/webhooks
```

```typescript
import { webhooks } from "@frontal-labs/webhooks";

const endpoint = await webhooks.endpoints.create({
  url: "https://my-app.com/webhooks/frontal",
  events: ["invoice.paid", "subscription.created"],
  secret: "whsec_..."
});
```

## Client Setup

```typescript
import { createWebhooksClient } from "@frontal-labs/webhooks";

const webhooks = createWebhooksClient({
  apiKey: process.env.FRONTAL_API_KEY!,
  baseUrl: "https://api.frontal.dev/v1"
});
```

## Endpoints

```typescript
// Create endpoint
const endpoint = await webhooks.endpoints.create({
  url: "https://my-app.com/webhooks/frontal",
  events: ["invoice.paid", "subscription.cancelled"],
  secret: "whsec_abc123",       // used for HMAC signing
  timeout: 5000,                 // ms before delivery fails
  retries: 3,
  retryDelay: 1000
});

// List endpoints
const endpoints = await webhooks.endpoints.list();

// Update
await webhooks.endpoints.update(endpoint.id, {
  events: ["invoice.paid", "subscription.created", "subscription.cancelled"]
});

// Delete
await webhooks.endpoints.delete(endpoint.id);
```

## Delivery

```typescript
// List deliveries
const deliveries = await webhooks.deliveries.list({
  endpointId: endpoint.id,
  status: "failed",
  limit: 20
});

// Get delivery details
const delivery = await webhooks.deliveries.get("del_abc123");
// delivery.attempts, delivery.responseStatus, delivery.responseBody, delivery.error

// Replay failed delivery
await webhooks.deliveries.replay("del_abc123");
```

## HMAC Verification (Server-side)

```typescript
import { verifyWebhookSignature } from "@frontal-labs/webhooks";

function handler(req: Request) {
  const signature = req.headers.get("X-Frontal-Signature")!;
  const body = await req.text();

  const verified = verifyWebhookSignature({
    body,
    signature,
    secret: process.env.WEBHOOK_SECRET!
  });

  if (!verified) {
    return new Response("Invalid signature", { status: 401 });
  }

  const event = JSON.parse(body);
  // process event...
}
```

## Stats

```typescript
const stats = await webhooks.stats.get({
  endpointId: endpoint.id,
  from: "2025-01-01",
  to: "2025-01-31"
});
// stats.total, stats.success, stats.failed, stats.avgLatency
```

## Best Practices

- **Always verify HMAC signatures** server-side before processing
- **Return 200 quickly** — process the event asynchronously to avoid timeout retries
- **Set `retries: 3` and `retryDelay: 1000`** as minimums; use exponential backoff for critical events
- **Use separate endpoints per event type** for independent scaling and debugging

## Common Pitfalls

- **Do not** expose the webhook secret in client-side code
- **Do not** return 4xx/5xx for handler errors — Frontal treats them as delivery failures and retries
- **Do not** ignore delivery failures — monitor `webhooks.deliveries.list({ status: "failed" })` regularly

## References

- `references/webhook-events.md` — Full list of webhook event types and payloads
- `references/webhook-security.md` — HMAC rotation, IP allowlisting, and secret management
