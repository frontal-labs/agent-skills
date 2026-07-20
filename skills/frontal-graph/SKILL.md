---
name: frontal-graph
description: Query and mutate the Frontal knowledge graph with @frontal-labs/graph — entity CRUD, graph traversal, semantic search, relationships, and time-travel history. Use when working with entities, graph queries, knowledge graphs, or entity relationships.
metadata:
  internal: true
  author: frontal-labs
  version: "1.0"
  category: sdk
---

# Frontal SDK: Graph

## When to Use

- Creating or querying business entities (customer, ticket, order, etc.)
- Traversing relationships between entities
- Running semantic or similarity search over graph entities
- Auditing entity history and reverting state changes
- Any task mentioning `@frontal-labs/graph`, graph entities, or knowledge graph

## Quick Start

```bash
bun add @frontal-labs/graph
```

```typescript
import { graph } from "@frontal-labs/graph";

const customers = await graph.query({
  entityType: "customer",
  conditions: { tier: "enterprise" },
  limit: 25
});

await graph.use("customer").addRelationship(
  "cust_123", "ticket_456", "opened_ticket", { weight: 1 }
);
```

## Client Setup

```typescript
import { createGraphClient } from "@frontal-labs/graph";

const graph = createGraphClient({
  apiKey: process.env.FRONTAL_API_KEY!,
  baseUrl: "https://api.frontal.dev/v1"
});
```

## Entity Accessor Pattern

Every entity type gets a typed accessor via `graph.use(entityType)`:

```typescript
const customer = graph.use("customer");
const ticket  = graph.use("ticket");

const c = await customer.get("cust_123");
await customer.create({ name: "Acme Corp", tier: "enterprise" });
await customer.update("cust_123", { tier: "premium" });
await customer.delete("cust_123");

const list = await customer.list({ limit: 50, offset: 0 });
```

## Query API

```typescript
const results = await graph.query({
  entityType: "customer",
  conditions: {
    tier: "enterprise",
    createdAfter: "2025-01-01"
  },
  include: ["support_tickets", "subscriptions"],  // eager-load relationships
  orderBy: "created_at",
  order: "desc",
  limit: 25,
  cursor: "next_page_token"
});
// results.entities — array of matching entities
// results.nextCursor — pagination cursor
```

## Relationships

```typescript
// Add relationship
await graph.use("customer").addRelationship(
  "cust_123",           // from entity ID
  "ticket_456",         // to entity ID
  "opened_ticket",      // relationship type
  { weight: 1, metadata: { channel: "email" } }
);

// Remove relationship
await graph.use("customer").removeRelationship(
  "cust_123", "ticket_456", "opened_ticket"
);

// List relationships
const rels = await graph.use("customer").relationships("cust_123");
```

## Graph Traversal

```typescript
// Neighbors (1-hop)
const neighbors = await graph.neighbors({
  entityId: "cust_123",
  direction: "outgoing",    // "incoming" | "both"
  relationshipTypes: ["opened_ticket", "upgraded_to"],
  limit: 20
});

// Shortest path
const path = await graph.path({
  from: "cust_123",
  to: "ticket_456",
  maxDepth: 5
});
// path.steps — array of { entityId, relationshipType }
```

## Semantic Search

```typescript
const results = await graph.semanticSearch({
  query: "enterprise customers with open billing issues",
  entityTypes: ["customer", "ticket"],
  limit: 10,
  threshold: 0.75
});
// results — array of { entity, score }
```

## Time Travel & History

```typescript
// Entity history
const history = await graph.use("customer").history("cust_123");
// history — array of { timestamp, changedFields, previous, current }

// Point-in-time query
const asOf = await graph.query({
  entityType: "customer",
  asOf: "2025-06-01T00:00:00Z"
});
```

## Best Practices

- **Use typed entity accessors** (`graph.use("customer")`) over raw queries when possible
- **Index entities** on frequently-queried fields via ontology migrations
- **Use `semanticSearch`** for exploratory queries; use `query` for precise filters
- **Keep relationship types immutable** — treat them as verbs in your domain model

## Common Pitfalls

- **Do not** create cycles in hierarchical relationships without a `maxDepth` guard
- **Do not** store large blobs in entity metadata — use `@frontal-labs/blob` and link by ID
- **Do not** query without `limit` on unbounded entity types — default is not enforced server-side

## References

- `references/graph-query-language.md` — Full query syntax, operators, and indexing
- `references/graph-relationships.md` — Relationship cardinality and traversal patterns
- `references/graph-history.md` — Time-travel semantics and audit integration
