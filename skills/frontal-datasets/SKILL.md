---
name: frontal-datasets
description: Manage versioned datasets with @frontal-labs/datasets — CRUD operations, versioning, import/export, and data quality checks. Use when working with structured datasets, data versioning, or dataset pipelines.
metadata:
  internal: true
  author: frontal-labs
  version: "1.0"
  category: sdk
---

# Frontal SDK: Datasets

## When to Use

- Creating, updating, or deleting datasets
- Managing dataset versions and snapshots
- Importing/exporting data between datasets
- Running data quality checks on datasets
- Any task mentioning `@frontal-labs/datasets` or dataset versioning

## Quick Start

```bash
bun add @frontal-labs/datasets
```

```typescript
import { datasets } from "@frontal-labs/datasets";

const dataset = await datasets.create({
  name: "customer-churn",
  description: "Monthly churn predictions"
});

await datasets.versions.create(dataset.id, {
  data: churnRecords,
  label: "v1"
});
```

## Client Setup

```typescript
import { createDatasetsClient } from "@frontal-labs/datasets";

const datasets = createDatasetsClient({
  apiKey: process.env.FRONTAL_API_KEY!,
  baseUrl: "https://api.frontal.dev/v1"
});
```

## CRUD

```typescript
// Create
const ds = await datasets.create({
  name: "customer-churn",
  description: "Monthly churn predictions",
  schema: { type: "object", properties: { ... } }
});

// List
const list = await datasets.list({ limit: 50 });

// Get
const fetched = await datasets.get("ds_abc123");

// Update
await datasets.update("ds_abc123", { description: "Updated description" });

// Delete
await datasets.delete("ds_abc123");
```

## Versions

```typescript
// Create version
const version = await datasets.versions.create("ds_abc123", {
  data: records,           // array of objects
  label: "v2"
});

// List versions
const versions = await datasets.versions.list("ds_abc123");

// Get version
const v = await datasets.versions.get("ds_abc123", "ver_xyz");

// Compare versions
const diff = await datasets.versions.compare("ds_abc123", "ver_old", "ver_new");
```

## Import/Export

```typescript
// Import from CSV/JSON
await datasets.import("ds_abc123", {
  format: "csv",
  source: fileBuffer,
  options: { delimiter: ",", hasHeader: true }
});

// Export
const exportBlob = await datasets.export("ds_abc123", {
  format: "json",
  version: "latest"
});
```

## Best Practices

- **Use descriptive labels** for versions (`"v2-churn-model"` not `"v2"`)
- **Validate schema before import** to catch format errors early
- **Compare versions** before deploying to production to understand drift

## Common Pitfalls

- **Do not** import datasets without validating row counts match expectations
- **Do not** delete datasets referenced by active pipelines — use archive instead

## References

- `references/datasets-quality.md` — Data quality checks, profiling, and anomaly detection
- `references/datasets-lineage.md` — Dataset lineage tracking and impact analysis
