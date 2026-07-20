---
name: frontal-ontology
description: Manage domain models, schemas, and migrations with @frontal-labs/ontology — validation, AI-powered generation, rules, mixins, and suggestions. Use when modeling domains, evolving schemas, or implementing ontology-driven applications.
metadata:
  internal: true
  author: frontal-labs
  version: "1.0"
  category: sdk
---

# Frontal SDK: Ontology

## When to Use

- Defining or validating domain models and schemas
- Running AI-powered ontology generation from natural language
- Managing schema migrations and rollback
- Applying rules and mixins to object types
- Accepting/rejecting AI-generated schema suggestions
- Any task mentioning `@frontal-labs/ontology`, domain models, or schema migrations

## Quick Start

```bash
bun add @frontal-labs/ontology
```

```typescript
import { ontology } from "@frontal-labs/ontology";

await ontology.validation.validatePayload({
  objectType: "Incident",
  payload: { severity: "high", service: "payments" }
});
```

## Client Setup

```typescript
import { createOntologyClient } from "@frontal-labs/ontology";

const ontology = createOntologyClient({
  apiKey: process.env.FRONTAL_API_KEY!,
  baseUrl: "https://api.frontal.dev/v1"
});
```

## Validation

```typescript
const result = await ontology.validation.validatePayload({
  objectType: "Incident",
  payload: { severity: "high" }
});
// result.valid — boolean
// result.errors — array of validation errors
```

## AI-Powered Generation

```typescript
const proposal = await ontology.engine.generate({
  description: "Model a billing dispute lifecycle with ownership and SLA states.",
  substrates: ["billing", "support"]
});
// proposal.objectTypes — proposed entity types
// proposal.relationshipTypes — proposed relationships

// Accept suggestion
await ontology.engine.accept(proposal.id);

// Reject suggestion
await ontology.engine.reject(proposal.id, { reason: "Conflicts with existing model" });
```

## Object Types

```typescript
// List object types
const types = await ontology.objects.listObjectTypes();

// Get object type details
const type = await ontology.objects.get("Incident");

// Create object type
const created = await ontology.objects.create({
  name: "Incident",
  description: "A production incident requiring investigation",
  properties: [
    { name: "severity", type: "enum", values: ["low", "medium", "high", "critical"] },
    { name: "service", type: "string" },
    { name: "owner", type: "reference", target: "User" }
  ]
});
```

## Relationships

```typescript
// List relationship types
const rels = await ontology.relationships.listTypes();

// Create relationship
await ontology.relationships.create({
  name: "owns",
  source: "Team",
  target: "Incident",
  cardinality: "one-to-many"
});
```

## Migrations

```typescript
// Create migration
const migration = await ontology.migrations.create({
  name: "add-sla-field",
  objectType: "Incident",
  changes: [
    { op: "add", property: { name: "slaDeadline", type: "datetime" } }
  ]
});

// Apply migration
await ontology.migrations.apply(migration.id);

// Rollback
await ontology.migrations.rollback(migration.id);
```

## Rules & Mixins

```typescript
// List rules
const rules = await ontology.rules.list();

// Create rule
await ontology.rules.create({
  name: "auto-assign-owner",
  objectType: "Incident",
  condition: "input.severity === 'critical'",
  action: "assign_owner(input.service.owner)"
});
```

## Best Practices

- **Use AI generation for initial proposals**, then review manually before accepting
- **Version migrations** and test rollback before applying to production
- **Keep object types focused** — split large types into smaller, composable ones
- **Use mixins** for cross-cutting concerns (audit, timestamps)

## Common Pitfalls

- **Do not** accept AI-generated proposals without domain review — they may miss business rules
- **Do not** create circular relationship types without explicit `maxDepth` guards
- **Do not** run migrations without a rollback plan

## References

- `references/ontology-migrations.md` — Migration syntax, safety checks, and rollback strategies
- `references/ontology-ai.md` — AI generation parameters, prompt engineering, and quality tuning
