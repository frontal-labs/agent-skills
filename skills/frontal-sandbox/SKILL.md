---
name: frontal-sandbox
description: Run isolated code execution with @frontal-labs/sandbox — compile-and-judge engine for running untrusted code in multiple languages with streaming, snapshots, and resource limits. Use when executing user code, running coding challenges, or implementing sandboxed evaluation.
metadata:
  internal: true
  author: frontal-labs
  version: "1.0"
  category: sdk
---

# Frontal SDK: Sandbox

## When to Use

- Running untrusted user code in isolated environments
- Implementing coding challenges, judges, or evaluation pipelines
- Executing code in multiple languages with resource limits
- Any task mentioning `@frontal-labs/sandbox`, code execution, or sandboxed evaluation

## Quick Start

```bash
bun add @frontal-labs/sandbox
```

```typescript
import { sandbox } from "@frontal-labs/sandbox";

const result = await sandbox.submit({
  language: "python",
  code: "print('Hello, World!')",
  judge: "default",
  cases: [{ stdin: "", expected: "Hello, World!\n" }]
});
```

## Client Setup

```typescript
import { createSandboxClient } from "@frontal-labs/sandbox";

const sandbox = createSandboxClient({
  apiKey: process.env.FRONTAL_API_KEY!,
  baseUrl: process.env.FRONTAL_SANDBOX_API_URL ?? "https://api.frontal.dev/v1"
});
```

## Languages

```typescript
const languages = await sandbox.languages();
// ["python", "javascript", "typescript", "go", "rust", "java", "cpp"]
```

## Self-Test

```typescript
const result = await sandbox.selfTest({
  language: "python",
  code: "print('ok')"
});
// result.compile — { success, errors }
// result.summary — { stdout, stderr, exitCode, durationMs }
```

## Submit (Judge)

```typescript
const result = await sandbox.submit({
  language: "python",
  code: `def solve(a, b): return a + b`,
  judge: "default",
  cases: [
    { stdin: "1 2", expected: "3" },
    { stdin: "10 20", expected: "30" }
  ],
  resourceLimits: {
    cpuMs: 5000,
    memoryMb: 256,
    diskMb: 100
  }
});
// result.compile — { success, errors }
// result.cases — array of { stdin, expected, actual, passed, durationMs }
// result.summary — { passed, total, durationMs }
```

## Metrics

```typescript
const metrics = await sandbox.metrics.get();
// metrics.totalExecutions, metrics.successRate, metrics.avgDurationMs
```

## Best Practices

- **Always set `resourceLimits`** — never rely on defaults for untrusted code
- **Use `selfTest`** before `submit` to catch syntax errors early
- **Prefer `judge: "default"`** unless you need custom evaluation logic
- **Stream results** for long-running submissions via async iteration

## Common Pitfalls

- **Do not** execute arbitrary code without resource limits — sandbox escape risk
- **Do not** use `selfTest` as a security boundary — it is for syntax checking only
- **Do not** assume `cases` run in order — parallel execution may happen

## References

- `references/sandbox-judges.md` — Built-in judges (default, interactive, diff)
- `references/sandbox-security.md` — Isolation guarantees, filesystem restrictions, network policy
