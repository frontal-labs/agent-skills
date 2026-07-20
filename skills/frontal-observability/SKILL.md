---
name: frontal-observability
description: Collect logs, metrics, and traces with @frontal-labs/observability — dashboards, alerts, OTLP export, and structured observability. Use when implementing monitoring, logging, distributed tracing, or alerting with Frontal.
metadata:
  internal: true
  author: frontal-labs
  version: "1.0"
  category: sdk
---

# Frontal SDK: Observability

## When to Use

- Emitting structured logs, metrics, or traces
- Creating dashboards and configuring alerts
- Exporting telemetry via OTLP
- Building observability into applications and services
- Any task mentioning `@frontal-labs/observability`, logs, metrics, traces, or alerts

## Quick Start

```bash
bun add @frontal-labs/observability
```

```typescript
import { observability } from "@frontal-labs/observability";

await observability.logs.info("Invoice processed", { invoiceId: "inv_123" });

await observability.metrics.increment("invoices.processed", { plan: "pro" });
```

## Client Setup

```typescript
import { createObservabilityClient } from "@frontal-labs/observability";

const observability = createObservabilityClient({
  apiKey: process.env.FRONTAL_API_KEY!,
  baseUrl: "https://api.frontal.dev/v1"
});
```

## Logs

```typescript
// Structured logging
await observability.logs.info("Payment succeeded", {
  invoiceId: "inv_123",
  customerId: "cust_456",
  amount: 2900
});

// With severity
await observability.logs.error("Payment gateway timeout", {
  gateway: "stripe",
  latencyMs: 30000
});

// Query logs
const logs = await observability.logs.query({
  query: "level:error AND service:billing",
  from: "2025-01-01T00:00:00Z",
  to: "2025-01-02T00:00:00Z",
  limit: 100
});
```

## Metrics

```typescript
// Counter
await observability.metrics.increment("api.requests", {
  method: "POST",
  path: "/v1/invoices",
  status: "201"
});

// Gauge
await observability.metrics.gauge("queue.depth", 42, {
  queue: "invoice-processing"
});

// Histogram
await observability.metrics.histogram("http.request.duration", 120, {
  method: "GET",
  path: "/v1/customers"
});
```

## Traces

```typescript
// Create span (auto-instrumented if using OpenTelemetry SDK)
const span = observability.traces.startSpan("process-invoice", {
  attributes: { invoiceId: "inv_123" }
});

try {
  await processInvoice();
  span.setStatus({ code: 1 });  // OK
} catch (err) {
  span.setStatus({ code: 2, message: err.message });  // ERROR
} finally {
  span.end();
}
```

## Alerts

```typescript
// Create alert
await observability.alerts.create({
  name: "High error rate on billing",
  condition: "rate(errors{service=billing}) > 0.05",
  duration: "5m",
  channels: ["email", "slack"],
  severity: "critical"
});

// List alerts
const alerts = await observability.alerts.list();
```

## Dashboards

```typescript
// Create dashboard
await observability.dashboards.create({
  name: "Billing Overview",
  panels: [
    { type: "graph", query: "sum(invoices_processed_total)", title: "Invoices" },
    { type: "gauge", query: "payment_gateway_latency_p99", title: "P99 Latency" }
  ]
});
```

## Best Practices

- **Use structured attributes** on every log line — never log unstructured blobs
- **Set trace context** at service boundaries to enable distributed tracing
- **Create alerts on error rate, not just count** — `rate()` prevents scaling noise
- **Use histograms** for latency — they preserve percentile data

## Common Pitfalls

- **Do not** log PII or secrets — use redaction in the SDK or filter at ingestion
- **Do not** create alerts without a runbook link
- **Do not** sample traces at 100% in production without understanding storage cost

## References

- `references/observability-otel.md` — OpenTelemetry integration and auto-instrumentation
- `references/observability-alerts.md` — Alert design patterns and severity guidelines
