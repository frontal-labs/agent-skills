---
name: frontal-agents
description: Build and operate AI agents with @frontal-labs/agents — define agents with the builder DSL, deploy, pause/resume, run executions, watch events via SSE, and manage rollouts. Use when creating AI agents, orchestrating agent executions, or implementing agent workflows with Frontal.
metadata:
  internal: true
  author: frontal-labs
  version: "1.0"
  category: sdk
---

# Frontal SDK: Agents

## When to Use

- Defining new AI agents with triggers, scopes, and memory config
- Deploying, pausing, resuming, or rolling back agent versions
- Running agent executions and watching live event streams
- Implementing agent-based ticket triage, support routing, or automation
- Any task mentioning `@frontal-labs/agents`, agent lifecycle, or agent execution

## Quick Start

```bash
bun add @frontal-labs/agents
```

```typescript
import { agents } from "@frontal-labs/agents";

const agent = await agents
  .define("support-triage")
  .description("Classifies and routes support tickets")
  .trigger("support.ticket.created")
  .tags("support", "triage")
  .create();

const run = await agents.use(agent.id).message("support.ticket.created", {
  ticketId: "t_987",
  text: "Payment failed after upgrade"
});

for await (const event of agents.use(agent.id).watch(run.id)) {
  console.log(event.type, event.data);
}
```

## Client Setup

```typescript
// Singleton
import { agents } from "@frontal-labs/agents";

// Factory
import { createAgentsClient } from "@frontal-labs/agents";
const agents = createAgentsClient({
  apiKey: process.env.FRONTAL_API_KEY!,
  baseUrl: process.env.FRONTAL_AGENTS_API_URL ?? process.env.FRONTAL_API_URL ?? "https://api.frontal.dev/v1"
});

// Composed
import { FrontalClient } from "@frontal-labs/core";
import { AgentsService } from "@frontal-labs/agents";
const core = new FrontalClient({ apiKey, baseUrl: "https://api.frontal.dev/v1" });
const agents = new AgentsService(core.httpClient);
```

## Agent Definition (Builder DSL)

```typescript
const agent = await agents
  .define("ticket-triager")
  .description("Routes support tickets to the right team")
  .trigger("support.ticket.created")           // event trigger
  // .schedule("0 */6 * * *")                 // cron trigger
  // .webhook("/webhooks/triage")              // webhook trigger
  .scope("support")                            // domain scope
  .confidence({ threshold: 0.8 })             // min confidence to auto-resolve
  .memory({ ttl: 3600, maxSize: 1000 })       // execution memory
  .rateLimit({ maxConcurrent: 5, queueSize: 20 })
  .timeout(30000)
  .tags("support", "automation")
  .create();
```

### Trigger Types

| Method | Trigger | Use Case |
|--------|---------|----------|
| `.trigger(eventName)` | Event-driven | React to platform events |
| `.schedule(cron)` | Cron-based | Periodic tasks |
| `.webhook(path)` | HTTP webhook | External integrations |
| `.manual()` | On-demand | Admin-triggered runs |

## Agent Operations

```typescript
// CRUD
const created = await agents.define("name").create();
const listed = await agents.list({ scope: "support" });
const fetched = await agents.get(created.id);
await agents.update(created.id).description("New desc").save();
await agents.delete(created.id);

// Lifecycle
await agents.deploy(created.id);           // deploy new version
await agents.pause(created.id);            // pause execution
await agents.resume(created.id);           // resume execution
await agents.rollback(created.id, "v2");   // rollback to version
await agents.simulate(created.id, { input }); // dry-run
```

## Execution

```typescript
// Fire-and-forget with input
const run = await agents.use(agentId).message("support.ticket.created", {
  ticketId: "t_987",
  text: "Payment failed"
});

// Watch execution events via async iteration (SSE)
for await (const event of agents.use(agentId).watch(run.id)) {
  console.log(event.type, event.data);
  // event.type: "started" | "step" | "tool_call" | "completed" | "failed"
}

// Poll to completion
const done = await agents.use(agentId).waitForCompletion(run.id);
// done.status — "completed" | "failed" | "cancelled"

// Read transcript
const transcript = await agents.use(agentId).conversation(run.id);
```

## Execution Object

```typescript
interface Execution {
  id: string;
  agentId: string;
  status: "queued" | "running" | "completed" | "failed" | "cancelled";
  input: Record<string, unknown>;
  output?: Record<string, unknown>;
  error?: string;
  startedAt: string;
  completedAt?: string;
}
```

## Best Practices

- **Scope agents** to a domain (`support`, `billing`) for observability and access control
- **Set `confidence.threshold`** to avoid low-confidence automated decisions
- **Use `.watch()`** for real-time observability in development; switch to `waitForCompletion()` in production
- **Tag agents** for filtering and cost attribution
- **Implement rate limiting** at the agent level, not just at the gateway

## Common Pitfalls

- **Do not** deploy an agent without first running `.simulate()` to validate behavior
- **Do not** set `maxConcurrent` higher than the downstream service can handle
- **Do not** ignore `Execution.output` — it may contain partial results even on `failed` status
- **Do not** call `.watch()` on a completed run — it throws; use `.conversation()` instead

## References

- `references/agent-patterns.md` — Common agent patterns (triage, escalation, approval)
- `references/agent-config.md` — Full configuration schema for confidence, memory, rate limits
- `references/agent-events.md` — SSE event types and payload shapes
