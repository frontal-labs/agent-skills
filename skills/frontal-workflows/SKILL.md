---
name: frontal-workflows
description: Orchestrate multi-step workflows with @frontal-labs/workflows — define approval flows, conditional logic, parallel branches, scheduled triggers, and webhook invocations. Use when building business processes, incident response playbooks, or multi-step approval chains with Frontal.
metadata:
  internal: true
  author: frontal-labs
  version: "1.0"
  category: sdk
---

# Frontal SDK: Workflows

## When to Use

- Building approval chains (manager approval, compliance review)
- Orchestrating multi-step business processes
- Implementing scheduled or event-triggered automation
- Creating incident response or onboarding playbooks
- Any task mentioning `@frontal-labs/workflows`, workflow DSL, or approval flows

## Quick Start

```bash
bun add @frontal-labs/workflows
```

```typescript
import { workflows } from "@frontal-labs/workflows";

const wf = await workflows
  .define("invoice-approval")
  .manual()
  .task("validate", { ruleset: "invoice-v1" })
  .approval("manager-signoff", ["finance@acme.com"])
  .task("post-ledger", { system: "erp" })
  .create();

const run = await workflows.use(wf.id).trigger({ invoiceId: "inv_001" });
await workflows.use(wf.id).waitForCompletion(run.id);
```

## Client Setup

```typescript
import { createWorkflowsClient } from "@frontal-labs/workflows";

const workflows = createWorkflowsClient({
  apiKey: process.env.FRONTAL_API_KEY!,
  baseUrl: "https://api.frontal.dev/v1"
});
```

## Workflow Definition (DSL)

### Triggers

```typescript
.manual()                                    // on-demand via API
.schedule("0 9 * * 1-5")                    // cron: weekdays at 9am
.event("billing.invoice.created")            // platform event
.webhook("/webhooks/invoice-approved")       // HTTP POST webhook
```

### Steps

| Method | Purpose | Example |
|--------|---------|---------|
| `.task(name, params)` | Execute a unit of work | `.task("send-email", { template: "welcome" })` |
| `.approval(name, approvers)` | Human approval gate | `.approval("manager-ok", ["manager@co.com"])` |
| `.condition(name, expr)` | Branch on expression | `.condition("is-enterprise", "input.plan === 'enterprise'")` |
| `.parallel(names)` | Fan-out to parallel steps | `.parallel(["notify-email", "notify-sms", "notify-slack"])` |
| `.delay(name, duration)` | Wait before proceeding | `.delay("grace-period", "24h")` |
| `.notification(name, channels)` | Send notification | `.notification("alert-ops", ["email", "slack"])` |

### Example: Conditional Approval Flow

```typescript
const wf = await workflows
  .define("expense-approval")
  .event("billing.expense.created")
  .task("validate-amount", { maxAmount: 10000 })
  .condition("needs-approval", "input.amount > 5000")
  .approval("director-approval", ["director@co.com"])
  .task("process-payment")
  .create();
```

## Execution

```typescript
// Trigger a run
const run = await workflows.use(wf.id).trigger({ expenseId: "exp_123" });

// Poll to completion
const done = await workflows.use(wf.id).waitForCompletion(run.id);
console.log(done.status, done.output);

// Or watch events in real-time
for await (const event of workflows.use(wf.id).watch(run.id)) {
  console.log(event.stepName, event.status);
}
```

## Approvals

```typescript
// List pending approvals
const pending = await workflows.approvals.list({ workflowId: wf.id });

// Approve
await workflows.approvals.approve(pending[0].id, {
  comment: "Looks good"
});

// Reject
await workflows.approvals.reject(pending[0].id, {
  comment: "Need more details"
});
```

## Templates

```typescript
// Create from template
const wf = await workflows.templates
  .get("incident-response")
  .create({ severity: "P1", service: "payments" });

// List available templates
const templates = await workflows.templates.list();
```

## Step Parameters

```typescript
interface TaskParams {
  handler: string;           // registered task handler name
  input?: Record<string, unknown>;
  timeout?: number;          // ms
  retries?: number;
  retryDelay?: number;
}

interface ApprovalParams {
  approvers: string[];       // emails or user IDs
  timeout?: number;          // auto-escalate after ms
  escalationPolicy?: string;
}
```

## Best Practices

- **Keep steps idempotent** — retries are automatic on transient failures
- **Use `.condition()` early** to avoid unnecessary parallel work
- **Set `timeout` on `.task()`** for long-running external calls
- **Use `.approval()` with `escalationPolicy`** for SLA-bound processes
- **Prefer `.watch()` during development** for visibility; use `waitForCompletion()` in production

## Common Pitfalls

- **Do not** chain `.task()` calls that modify the same resource without approval
- **Do not** omit `timeout` on `.task()` — it defaults to the workflow-level timeout
- **Do not** use `.parallel()` for steps with shared mutable state
- **Do not** call `.trigger()` on a `.paused()` workflow — it throws `ForbiddenError`

## References

- `references/workflow-dsl.md` — Complete DSL reference with all chainable methods
- `references/workflow-templates.md` — Built-in templates and how to create custom ones
- `references/workflow-events.md` — Watch event schema, SSE connection lifecycle
