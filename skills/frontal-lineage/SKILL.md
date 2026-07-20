---
name: frontal-lineage
description: Trace data lineage and dependencies with @frontal-labs/lineage — upstream/downstream impact analysis, column-level lineage, and data flow visualization. Use when analyzing data pipelines, impact analysis, or compliance audits.
metadata:
  internal: true
  author: frontal-labs
  version: "1.0"
  category: sdk
---

# Frontal SDK: Lineage

## When to Use

- Tracing upstream and downstream data dependencies
- Performing impact analysis before schema changes
- Generating column-level lineage for compliance audits
- Visualizing data flows across pipelines and datasets
- Any task mentioning `@frontal-labs/lineage`, data lineage, or impact analysis

## Quick Start

```bash
bun add @frontal-labs/lineage
```

```typescript
import { lineage } from "@frontal-labs/lineage";

const impact = await lineage.analyze("dataset:customer-churn");
// impact.downstream — all datasets, tables, and endpoints affected
```

## Client Setup

```typescript
import { createLineageClient } from "@frontal-labs/lineage";

const lineage = createLineageClient({
  apiKey: process.env.FRONTAL_API_KEY!,
  baseUrl: "https://api.frontal.dev/v1"
});
```

## Graph & Impact

```typescript
// Build lineage graph for an asset
const graph = await lineage.graph.build({
  asset: "dataset:customer-churn",
  depth: 3,                // hops to traverse
  direction: "both"        // "upstream" | "downstream" | "both"
});
// graph.nodes — assets
// graph.edges — transformations between assets

// Impact analysis
const impact = await lineage.analyze({
  asset: "dataset:customer-churn",
  change: { type: "schema", field: "email" }
});
// impact.downstream — affected assets
// impact.severity — "low" | "medium" | "high"
```

## Column-Level Lineage

```typescript
// Trace a specific column
const columnLineage = await lineage.column("customer-churn", "email");
// columnLineage.path — array of { asset, transformation, column }
// columnLineage.sources — raw source systems
```

## Export

```typescript
// Export as JSON for visualization
const export_ = await lineage.export("dataset:customer-churn", {
  format: "json",
  depth: 3
});

// Export as Mermaid diagram
const mermaid = await lineage.export("dataset:customer-churn", {
  format: "mermaid"
});
```

## Best Practices

- **Run impact analysis before schema changes** — `analyze()` returns severity
- **Use column-level lineage** for GDPR/CCPA data mapping
- **Cache lineage graphs** — they are expensive to build for large datasets

## Common Pitfalls

- **Do not** skip impact analysis for "minor" field additions — downstream consumers may break
- **Do not** rely on lineage graphs for real-time data — they are computed from pipeline metadata
- **Do not** export lineage for the entire platform in one call — scope to a domain

## References

- `references/lineage-visualization.md` — Embedding lineage diagrams in UIs
- `references/lineage-automation.md` — Automated impact analysis in CI/CD
