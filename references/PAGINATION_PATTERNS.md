# Pagination Patterns Reference

## PageResult<T>

All list endpoints return `PageResult<T>`:

```typescript
interface PageResult<T> {
  data: T[];
  meta: {
    requestId: string;
    timestamp: string;
    version: string;
  };
  pagination: {
    cursor?: string;     // for cursor-based pagination
    offset?: number;     // for offset-based pagination
    limit: number;
    hasMore: boolean;
    total?: number;      // only present when requested
  };
}
```

## Consumption Patterns

### Async Iteration (preferred)

```typescript
const page = await sdk.billing.subscriptions.list({ limit: 50 });
for await (const sub of page) {
  console.log(sub.id);
}
```

### Manual Pagination

```typescript
let cursor: string | undefined;
do {
  const page = await sdk.billing.subscriptions.list({ limit: 50, cursor });
  processPage(page.data);
  cursor = page.pagination.hasMore ? page.pagination.cursor : undefined;
} while (cursor);
```

### Collect All

```typescript
const all = await page.all();  // internally iterates all pages
```

### First Item

```typescript
const first = await page.first();  // returns T | undefined
```

### Count

```typescript
const count = await page.count();  // returns total if server provides it
```

## Cursor vs. Offset

- **Cursor-based**: Used for real-time data (events, logs). `hasMore` + `cursor`.
- **Offset-based**: Used for static datasets. `offset` + `limit` + `total`.

## Backpressure

When consuming large result sets, use async iteration instead of `.all()` to avoid
loading everything into memory:

```typescript
const stream = sdk.billing.subscriptions.listStream({ batchSize: 100 });
for await (const batch of stream) {
  await processBatch(batch);
}
```
