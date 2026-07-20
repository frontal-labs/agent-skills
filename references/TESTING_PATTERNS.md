# Testing Patterns Reference

## @frontal-labs/testing

The `@frontal-labs/testing` package provides shared test utilities:

```typescript
import {
  createTestClient,
  createMockFetch,
  mockPageResponse,
  mockErrorResponse,
  integrationPage
} from "@frontal-labs/testing";
```

## Mocking HTTP Calls

```typescript
const mockFetch = createMockFetch();

mockFetch.mockResponseOnce("GET", "/v1/subscriptions", {
  subscriptions: [{ id: "sub_1", plan: "pro" }],
  pagination: { total: 1, limit: 20, offset: 0, hasMore: false }
});

const client = createTestClient({ mockFetch });
const result = await client.billing.subscriptions.list();
expect(result.data).toHaveLength(1);
```

## Integration Harness

```typescript
import { createIntegrationHarness } from "@frontal-labs/testing";

const harness = await createIntegrationHarness({
  apiKey: process.env.FRONTAL_API_KEY!,
  baseUrl: "https://api.frontal.dev/v1"
});

// Use harness.ai, harness.billing, etc.
const subs = await harness.billing.subscriptions.list();
```

## Fixtures

```typescript
import { fixtures } from "@frontal-labs/testing";

const customer = fixtures.customer({ tier: "enterprise" });
const invoice = fixtures.invoice({ customerId: customer.id, amount: 10000 });
```

## Best Practices

- **Use `createTestClient`** for unit tests — it captures requests for assertion
- **Use `createIntegrationHarness`** only in integration tests with real API keys
- **Mock at the service boundary** — never mock internal SDK methods
- **Test error paths** with `mockErrorResponse(404)`, `mockErrorResponse(429)`, etc.

## Common Pitfalls

- **Do not** use `vitest.mock("@frontal-labs/core")` — it breaks internals; use `createTestClient` instead
- **Do not** share mock state between tests — create fresh `createMockFetch()` per test
- **Do not** run integration tests in CI without `FRONTAL_API_KEY` — gate them behind an env flag
