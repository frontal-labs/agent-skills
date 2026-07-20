---
name: frontal-governance
description: Enforce policies, manage RBAC, and run compliance checks with @frontal-labs/governance. Use when implementing access control, policy engines, compliance reporting, or governance workflows with Frontal.
metadata:
  internal: true
  author: frontal-labs
  version: "1.0"
  category: sdk
---

# Frontal SDK: Governance

## When to Use

- Defining and enforcing organizational policies
- Managing roles, permissions, and access-control checks
- Running compliance assessments and tracking violations
- Implementing approval workflows for sensitive operations
- Any task mentioning `@frontal-labs/governance`, policies, RBAC, compliance, or access control

## Quick Start

```bash
bun add @frontal-labs/governance
```

```typescript
import { governance } from "@frontal-labs/governance";

// Check access before an action
const allowed = await governance.accessControl.check({
  actorId: "user_123",
  action: "invoice:delete",
  resource: "inv_abc"
});

if (!allowed.allowed) {
  throw new Error(`Denied: ${allowed.reason}`);
}
```

## Client Setup

```typescript
import { createGovernanceClient } from "@frontal-labs/governance";

const governance = createGovernanceClient({
  apiKey: process.env.FRONTAL_API_KEY!,
  baseUrl: "https://api.frontal.dev/v1"
});
```

## Service Namespaces

```typescript
governance.policies         // Policy CRUD, validation, evaluation
governance.compliance       // Frameworks, assessments, violations, scoring
governance.accessControl    // RBAC: roles, permissions, access checks
```

## Policies

```typescript
// List policies
const policies = await governance.policies.list({
  category: "data-classification",
  status: "active"
});

// Create policy
const policy = await governance.policies.create({
  name: "PII Encryption Required",
  description: "All PII fields must be encrypted at rest",
  category: "data-classification",
  effect: "deny",
  condition: {
    field: "data_classification",
    operator: "equals",
    value: "pii"
  },
  actions: ["*:read", "*:write"],
  resources: ["*"]
});

// Validate policy syntax
const validation = await governance.policies.validate({
  expression: "resource.data_classification == 'pii' && actor.role != 'admin'"
});

// Evaluate policy against a context
const result = await governance.policies.evaluate({
  policyId: "pol_123",
  context: {
    actor: { id: "user_123", role: "analyst" },
    resource: { id: "doc_456", data_classification: "pii" },
    action: "documents:read"
  }
});
// result.allowed — boolean
// result.matchedPolicies — array of matching policy IDs
```

## RBAC: Roles & Permissions

```typescript
// List roles
const roles = await governance.accessControl.roles.list();

// Create role
const role = await governance.accessControl.roles.create({
  name: "billing-analyst",
  permissions: ["billing:invoices:read", "billing:subscriptions:read"]
});

// List permissions
const perms = await governance.accessControl.permissions.list();

// Create permission
const perm = await governance.accessControl.permissions.create({
  name: "billing:invoices:refund",
  description: "Issue refunds on invoices",
  resource: "billing:invoices",
  actions: ["refund"]
});

// Access check
const check = await governance.accessControl.check({
  actorId: "user_123",
  action: "billing:invoices:refund",
  resource: "inv_abc"
});
// check.allowed — boolean
// check.reason — explanation if denied
```

## Compliance

```typescript
// List frameworks
const frameworks = await governance.compliance.frameworks.list();
// SOC2, HIPAA, GDPR, PCI-DSS, etc.

// Create assessment
const assessment = await governance.compliance.assessments.create({
  frameworkId: "soc2",
  scope: ["billing", "storage"],
  assessorId: "user_456"
});

// List violations
const violations = await governance.compliance.violations.list({
  frameworkId: "soc2",
  severity: "high",
  status: "open"
});

// Resolve violation
await governance.compliance.violations.resolve("viol_789", {
  resolution: "Encryption key rotated",
  remediation: "none"
});

// Compliance score
const score = await governance.compliance.score({
  frameworkId: "soc2"
});
// score.overall — 0-100
// score.controls — per-control pass/fail
```

## Best Practices

- **Enforce access checks at the resource layer** — never trust client-side checks
- **Use policy templates** for common patterns (PII, PCI, HIPAA)
- **Run `policies.validate()`** before deploying to catch syntax errors
- **Review violations weekly** — set up alerts for `high` severity

## Common Pitfalls

- **Do not** use `effect: "allow"` policies without explicit deny rules — default-deny is safer
- **Do not** evaluate policies in a loop for bulk operations — use batch evaluation
- **Do not** store secrets in policy conditions — use references to secret management

## References

- `references/governance-policy-language.md` — Policy expression syntax, operators, functions
- `references/governance-templates.md` — Built-in policy templates for common compliance needs
- `references/governance-webhooks.md` — Violation and assessment event webhooks
