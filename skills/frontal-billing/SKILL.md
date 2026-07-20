---
name: frontal-billing
description: Manage subscriptions, invoices, payment methods, metering, and credits with @frontal-labs/billing. Use when implementing SaaS billing, subscription management, invoicing, usage metering, or payment flows with Frontal.
metadata:
  internal: true
  author: frontal-labs
  version: "1.0"
  category: sdk
---

# Frontal SDK: Billing

## When to Use

- Creating or managing subscription plans
- Handling invoices and payment collection
- Tracking usage metering for metered billing
- Managing payment methods and credits
- Any task mentioning `@frontal-labs/billing`, subscriptions, invoices, or metering

## Quick Start

```bash
bun add @frontal-labs/billing
```

```typescript
import { billing } from "@frontal-labs/billing";

const plans = await billing.plans.list();
const sub = await billing.subscriptions.create({
  customerId: "cust_123",
  planId: "plan_pro"
});
```

## Client Setup

```typescript
import { createBillingClient } from "@frontal-labs/billing";

const billing = createBillingClient({
  apiKey: process.env.FRONTAL_API_KEY!,
  baseUrl: "https://api.frontal.dev/v1"
});
```

## Service Namespaces

```typescript
billing.plans          // Plan CRUD, pricing, features
billing.subscriptions  // Subscription lifecycle, upgrades, downgrades, cancellations
billing.invoices        // Invoice generation, PDF, payment status
billing.paymentMethods  // Cards, bank accounts, payment intent flow
billing.metering        // Usage event ingestion, aggregation, rollup
billing.credits         // Credit balance, top-up, expiration
```

## Plans

```typescript
// List all plans
const plans = await billing.plans.list();

// Get a specific plan
const plan = await billing.plans.get("plan_pro");

// Create a plan
const created = await billing.plans.create({
  name: "Pro",
  interval: "month",
  price: 2900,          // cents
  currency: "USD",
  features: ["seats_5", "api_access"]
});
```

## Subscriptions

```typescript
// Create subscription
const sub = await billing.subscriptions.create({
  customerId: "cust_123",
  planId: "plan_pro",
  trialDays: 14
});

// Update subscription (upgrade/downgrade)
await billing.subscriptions.update(sub.id, {
  planId: "plan_enterprise",
  prorate: true
});

// Cancel
await billing.subscriptions.cancel(sub.id, { immediate: false });

// List for a customer
const subs = await billing.subscriptions.list({ customerId: "cust_123" });
```

## Invoices

```typescript
// List invoices
const invoices = await billing.invoices.list({
  customerId: "cust_123",
  status: "paid",       // draft | open | paid | void | uncollectible
  limit: 20
});

// Get invoice details
const invoice = await billing.invoices.get("inv_abc123");

// Invoice lines
const lines = await billing.invoices.listLines("inv_abc123");
```

## Payment Methods

```typescript
// Attach payment method
const pm = await billing.paymentMethods.create({
  customerId: "cust_123",
  type: "card",
  token: "pmt_xxx"      // from frontend (Stripe Elements, etc.)
});

// List payment methods
const methods = await billing.paymentMethods.list({ customerId: "cust_123" });

// Set default
await billing.paymentMethods.setDefault("cust_123", pm.id);
```

## Metering

```typescript
// Report usage event
await billing.metering.report({
  customerId: "cust_123",
  metric: "api_calls",
  quantity: 1500,
  timestamp: new Date().toISOString()
});

// Batch report
await billing.metering.reportBatch([
  { customerId: "cust_123", metric: "api_calls", quantity: 1500 },
  { customerId: "cust_123", metric: "storage_gb", quantity: 42 }
]);

// Usage summary
const usage = await billing.metering.getUsage({
  customerId: "cust_123",
  metric: "api_calls",
  from: "2025-01-01",
  to: "2025-01-31"
});
```

## Credits

```typescript
// Check balance
const balance = await billing.credits.getBalance("cust_123");

// Apply credit
await billing.credits.apply({
  customerId: "cust_123",
  amount: 5000,          // cents
  reason: "Compensation for outage"
});
```

## Best Practices

- **Always meter at the source** — call `billing.metering.report()` close to where usage occurs
- **Use `prorate: true`** when changing plans mid-cycle to avoid billing disputes
- **Set `trialDays`** on subscription creation to improve conversion
- **Listen to webhooks** for `invoice.paid`, `invoice.payment_failed` events to update UI

## Common Pitfalls

- **Do not** create invoices manually — use the subscription lifecycle to generate them
- **Do not** store payment method tokens client-side — use the frontend SDK to get a one-time token
- **Do not** meter in batch at end-of-day — real-time metering prevents surprise bills

## References

- `references/billing-webhooks.md` — Invoice and subscription webhook events
- `references/billing-tax.md` — Tax calculation and VAT handling
