# SDK Architecture Reference

## Package Hierarchy

```
@frontal-labs/core          Foundation: HTTP client, auth, retries, errors, pagination, circuit breaker
@frontal-labs/testing       Test mocks, fixtures, integration harnesses
@frontal-labs/sdk           Unified Sdk class aggregating all services
```

All domain packages depend on `@frontal-labs/core` only. They never depend on each other
except through the `Sdk` aggregator.

## Monorepo Structure (`/sdk-typescript`)

```
packages/
├── core/                    # Shared transport primitives
│   ├── src/
│   │   ├── index.ts         # Public API
│   │   ├── client.ts        # FrontalClient class
│   │   ├── http.ts          # HttpClient implementation
│   │   ├── errors.ts        # Typed error classes
│   │   ├── retry.ts         # Retry strategies (exponential, linear)
│   │   ├── circuit.ts       # CircuitBreaker implementation
│   │   ├── poll.ts          # pollUntil utility
│   │   ├── types.ts         # APIResponse, PageResult, QueryBuilder
│   │   └── schemas.ts       # Zod schemas for validation
│   ├── tests/
│   └── package.json
├── sdk/                     # Unified aggregator
├── ai/
├── agents/
├── workflows/
├── pipelines/
├── graph/
├── ontology/
├── billing/
├── auth/
├── blob/
├── governance/
├── audit/
├── events/
├── webhooks/
├── observability/
├── integrations/
├── sandbox/
├── workers/
├── datasets/
├── lineage/
├── schedules/
├── connectors/
└── testing/                 # Shared test utilities
```

## Build System

- **Turborepo** manages task orchestration with dependency-aware pipelines
- **Bun** is the package manager, runtime, and test runner
- **tsup** bundles ESM + CJS outputs
- **TypeScript** emits declarations via `tsc --emitDeclarationOnly`
- **Vitest** runs unit/integration tests
- **Biome** handles lint/format
- **Changesets** manages versioning and publishing

## Key Design Decisions

1. **Zod-first types**: All input/output types are derived from Zod schemas for runtime validation
2. **Lazy singletons**: Default instances use `Proxy` for lazy initialization to avoid allocating HTTP clients until first use
3. **Dual client modes**: Every service can be created from a `FrontalClient` instance OR standalone with a config object
4. **HTTP abstraction**: All services depend on `HttpClient` from core, which wraps `fetch` with retries, circuit breaking, and error mapping
5. **Tree-shakeable**: Individual service singletons are re-exported from `@frontal-labs/sdk` for optimal bundling
