---
name: frontal-events
description: Publish and subscribe to events with @frontal-labs/events — pub/sub event bus, dead-letter queues, schema registry, and client-side buffering. Use when building event-driven architectures or integrating with the Frontal event bus.
metadata:
  internal: true
  author: frontal-labs
  version: "1.0"
  category: sdk
---

# Frontal SDK: Events

## When to Use

- Publishing domain events to the Frontal event bus
- Subscribing to event streams with filtering
- Managing dead-letter queues and retry policies
- Registering and validating event schemas
- Any task mentioning `@frontal-labs/events`, event bus, pub/sub, or dead-letter queues

## Quick Start

```bash
bun add @frontal-labs/events
```

```typescript
import { events } from "@frontal-labs/events";

await events.publish("billing.invoice.created", {
  invoiceId: "inv_123",
  customerId: "cust_456",
  amount: 2900
});
```

## Client Setup

```typescript
import { createEventsClient } from "@frontal-labs/events";

const events = createEventsClient({
  apiKey: process.env.FRONTAL_API_KEY!,
  baseUrl: "https://api.frontal.dev/v1"
});
```

## Publishing

```typescript
// Single event
await events.publish("billing.invoice.created", {
  invoiceId: "inv_123",
  amount: 2900
});

// Batch publish
await events.publishBatch([
  { event: "billing.invoice.created", data: { invoiceId: "inv_123" } },
  { event: "billing.invoice.paid", data: { invoiceId: "inv_123" } }
]);

// With options
await events.publish("support.ticket.created", payload, {
  key: "ticket_123",           // idempotency key
  delay: 5000,                 // ms delay before delivery
  headers: { "X-Source": "web" }
});
```

## Subscriptions

```typescript
// Create subscription
const sub = await events.subscriptions.create({
  name: "invoice-processor",
  filter: {
    event: "billing.invoice.created",
    "data.amount": { $gte: 1000 }
  },
  handler: "https://my-app.com/webhooks/events",
  active: true
});

// List subscriptions
const subs = await events.subscriptions.list();

// Pause/resume
await events.subscriptions.pause(sub.id);
await events.subscriptions.resume(sub.id);
```

## Dead-Letter Queue

```typescript
// List DLQ messages
const messages = await events.deadLetter.list({
  subscriptionId: sub.id,
  status: "failed"
});

// Replay
await events.deadLetter.replay("dlq_msg_abc");
// Or replay all
await events.deadLetter.replayAll({ subscriptionId: sub.id });
```

## Schema Registry

```typescript
// Register schema
await events.schemas.register({
  event: "billing.invoice.created",
  version: "1.0.0",
  schema: {
    type: "object",
    properties: {
      invoiceId: { type: "string" },
      amount: { type: "integer" }
    },
    required: ["invoiceId", "amount"]
  }
});

// Validate payload
const valid = await events.schemas.validate({
  event: "billing.invoice.created",
  payload: { invoiceId: "inv_123", amount: 2900 }
});
```

## Best Practices

- **Use idempotency keys** for all financial events to prevent duplicates
- **Set filters on subscriptions** to reduce unnecessary delivery
- **Monitor DLQ** — automated replay only for transient failures; investigate root cause for schema errors
- **Version schemas** and deprecate old versions gracefully

## Common Pitfalls

- **Do not** publish events synchronously in request paths — use async fire-and-forget or background jobs
- **Do not** replay DLQ messages without first fixing the underlying handler bug
- **Do not** use broad wildcard filters (`*`) — they cause thundering-herd delivery

## References

- `references/events-filtering.md` — Advanced filter syntax and performance tips
- `references/events-ordering.md` — Event ordering guarantees and partitioning
