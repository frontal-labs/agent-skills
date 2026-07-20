# Error Handling Reference

## Error Class Hierarchy

```
FrontalError (base)
├── NotFoundError          404
├── UnauthorizedError      401
├── ForbiddenError         403
├── ValidationError        422
├── ConflictError          409
├── RateLimitError         429
├── ServiceError           500-599
├── NetworkError            -
└── TimeoutError            -
```

## Error Properties

```typescript
interface FrontalError extends Error {
  readonly name: string;           // e.g. "NotFoundError"
  readonly code?: string;          // API error code, e.g. "RESOURCE_NOT_FOUND"
  readonly context?: Record<string, unknown>;
  readonly requestId?: string;     // from ResponseMeta
}

interface RateLimitError extends FrontalError {
  readonly retryAfter?: number;    // ms until retry is safe
}

interface ErrorResponse {
  error: string;
  code: string;
  details?: ErrorField[];
  requestId: string;
  docs?: string;
}
```

## Recovery Strategies

| Error | Strategy |
|-------|----------|
| `RateLimitError` | Wait `retryAfter` ms, then retry with backoff |
| `ServiceError` (502/503) | Exponential backoff, max 3 retries |
| `NetworkError` | Retry with jitter, check connectivity |
| `TimeoutError` | Increase timeout or split request |
| `ValidationError` | Fix request payload — do not retry |
| `NotFoundError` | Graceful degradation — resource may not exist |
| `ForbiddenError` | Check token scopes, escalate permissions |

## Circuit Breaker States

```
CLOSED → (failureThreshold breaches) → OPEN → (resetTimeout passes) → HALF_OPEN
                                                              ↓
                                                    (success) → CLOSED
                                                    (failure) → OPEN
```

## Best Practices

- Always use `instanceof FrontalError` to type-check before recovery
- Log `requestId` from `error.context?.requestId` for support correlation
- Never swallow `FrontalError` silently — at minimum, log the `name` and `message`
- Use `parseFrontalError(err)` when catching unknown errors to normalize them
