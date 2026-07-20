---
name: frontal-storage
description: Manage object storage with @frontal-labs/blob — upload, download, stream, list, signed URLs, and metadata. Use when handling files, artifacts, media, or any blob/object storage with Frontal.
metadata:
  internal: true
  author: frontal-labs
  version: "1.0"
  category: sdk
---

# Frontal SDK: Storage (Blob)

## When to Use

- Uploading or downloading files, images, videos, or artifacts
- Generating signed URLs for client-side uploads/downloads
- Streaming large files without loading into memory
- Managing object metadata, copying, or moving storage objects
- Any task mentioning `@frontal-labs/blob`, blob storage, or object storage

## Quick Start

```bash
bun add @frontal-labs/blob
```

```typescript
import { blob } from "@frontal-labs/blob";

await blob.upload("report.pdf", fileBuffer, {
  contentType: "application/pdf",
  metadata: { generatedBy: "billing" }
});

const url = await blob.getSignedUrl("report.pdf", { expiresIn: 3600 });
```

## Client Setup

```typescript
import { createBlobClient } from "@frontal-labs/blob";

const blob = createBlobClient({
  apiKey: process.env.FRONTAL_API_KEY!,
  baseUrl: "https://api.frontal.dev/v1"
});
```

## Upload

```typescript
// From Buffer
await blob.upload("documents/report.pdf", buffer, {
  contentType: "application/pdf",
  metadata: { author: "alice", department: "finance" }
});

// From ReadableStream
await blob.upload("videos/demo.mp4", readableStream, {
  contentType: "video/mp4"
});

// With custom content type and cache control
await blob.upload("images/logo.png", pngBuffer, {
  contentType: "image/png",
  cacheControl: "public, max-age=31536000"
});
```

## Download

```typescript
// Download as Buffer
const file = await blob.download("documents/report.pdf");
// file.buffer — Buffer
// file.contentType — "application/pdf"
// file.size — bytes
// file.metadata — user-provided metadata

// Download as stream (for large files)
const stream = await blob.downloadStream("videos/demo.mp4");
stream.pipe(response);
```

## Signed URLs

```typescript
// Read URL (download)
const readUrl = await blob.getSignedUrl("documents/report.pdf", {
  expiresIn: 3600     // seconds
});

// Write URL (upload) — for client-side direct upload
const writeUrl = await blob.getSignedUploadUrl("uploads/avatar.png", {
  contentType: "image/png",
  expiresIn: 300
});
```

## List and Metadata

```typescript
// List objects
const objects = await blob.list({
  prefix: "documents/",
  limit: 100
});
// objects — array of { key, size, contentType, lastModified, metadata }

// Get metadata
const meta = await blob.getMetadata("documents/report.pdf");
```

## Delete, Copy, Move

```typescript
// Delete
await blob.delete("documents/report.pdf");

// Batch delete
await blob.deleteBatch(["a.pdf", "b.pdf", "c.pdf"]);

// Copy
await blob.copy("old/path.pdf", "new/path.pdf");

// Move (copy + delete)
await blob.move("old/path.pdf", "archive/path.pdf");
```

## Best Practices

- **Use `downloadStream`** for files >10MB to avoid memory pressure
- **Set `cacheControl`** on immutable assets for CDN performance
- **Use signed URLs** for client-side uploads — never expose the API key in the browser
- **Organize with prefixes** (`uploads/`, `artifacts/`, `exports/`) for lifecycle policies

## Common Pitfalls

- **Do not** generate signed URLs with long expiry (max recommended: 24h)
- **Do not** use `upload` for files >50MB — use multipart upload via `uploadStream`
- **Do not** rely on `list()` without `prefix` — it scans the entire bucket
- **Do not** delete objects without checking `metadata` — it may be referenced by graph entities

## References

- `references/storage-lifecycle.md` — Retention policies, cleanup rules, and lifecycle hooks
- `references/storage-cdn.md` — CDN integration, cache invalidation, and signed cookie setup
