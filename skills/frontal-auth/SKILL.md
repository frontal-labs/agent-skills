---
name: frontal-auth
description: Authenticate users and manage identity with @frontal-labs/auth — sign-up/sign-in, MFA, OAuth providers, SSO, session management, and admin user operations. Use when implementing login flows, user management, or identity features with Frontal.
metadata:
  internal: true
  author: frontal-labs
  version: "1.0"
  category: sdk
---

# Frontal SDK: Auth

## When to Use

- Implementing sign-up, sign-in, and password reset flows
- Configuring MFA, OAuth, or SSO providers
- Managing users, roles, and permissions via admin API
- Handling sessions, tokens, and refresh flows
- Any task mentioning `@frontal-labs/auth`, authentication, MFA, OAuth, or SSO

## Quick Start

```bash
bun add @frontal-labs/auth
```

```typescript
import { auth } from "@frontal-labs/auth";

const session = await auth.signIn({
  email: "user@acme.com",
  password: "secure-password"
});
// session.access_token — JWT for API calls
// session.user — { id, email, emailConfirmedAt }
```

## Client Setup

```typescript
import { createAuthClient } from "@frontal-labs/auth";

const auth = createAuthClient({
  apiKey: process.env.FRONTAL_API_KEY!,
  baseUrl: "https://api.frontal.dev/v1"
});
```

## Authentication Flows

### Email + Password

```typescript
// Sign up
const user = await auth.signUp({
  email: "user@acme.com",
  password: "secure-password",
  options: {
    data: { fullName: "Alice" }
  }
});

// Sign in
const session = await auth.signIn({
  email: "user@acme.com",
  password: "secure-password"
});

// Sign out
await auth.signOut();
```

### OAuth

```typescript
// Get OAuth URL
const url = await auth.getOAuthUrl("google");

// Exchange code for session
const session = await auth.exchangeCodeForSession("code_from_redirect");

// Supported providers: google, github, gitlab, azure, okta, saml
```

### MFA

```typescript
// Enroll MFA
const factor = await auth.mfa.enroll({
  factorType: "totp",
  friendlyName: "Alice's phone"
});
// factor.totp.secret — base32 secret for QR code
// factor.totp.qr_code — data URL for QR image

// Verify and challenge
const challenge = await auth.mfa.challenge({
  factorId: factor.id,
  type: "totp"
});

// Verify challenge
const verified = await auth.mfa.verify({
  factorId: factor.id,
  challengeId: challenge.id,
  code: "123456"
});
```

### SSO (SAML/OIDC)

```typescript
// Initiate SSO
const ssoUrl = await auth.sso.getSsoUrl("acme-corp");

// Unlink SSO
await auth.sso.unlink({ provider: "saml", ssoProviderId: "acme-corp" });
```

## Session Management

```typescript
// Refresh session
const refreshed = await auth.refreshSession({
  refresh_token: session.refresh_token
});

// Get current session
const current = await auth.getSession();

// Update user
await auth.updateUser({
  email: "new@acme.com",
  password: "new-password"
});

// Reset password (request)
await auth.resetPasswordForEmail("user@acme.com");
```

## Admin APIs

```typescript
// List users
const users = await auth.admin.listUsers({
  page: 1,
  perPage: 10,
  email: "alice@acme.com"
});

// Create user (admin)
const user = await auth.admin.createUser({
  email: "bob@acme.com",
  password: "temp-pass",
  emailConfirm: true
});

// Delete user
await auth.admin.deleteUser(user.id);
```

## Best Practices

- **Never store passwords** — always use the auth SDK methods
- **Use MFA for admin accounts** — enforce via policy in governance SDK
- **Refresh tokens before expiry** — implement a client-side refresh loop
- **Use SSO for enterprise** — map SSO groups to governance roles for RBAC

## Common Pitfalls

- **Do not** expose `apiKey` in client-side code — use anonymous auth or a backend proxy
- **Do not** skip email confirmation in production unless using SSO
- **Do not** store `access_token` in localStorage — use httpOnly cookies or secure storage
- **Do not** call admin APIs from the browser — restrict to server-side

## References

- `references/auth-providers.md` — OAuth provider setup, SAML configuration
- `references/auth-policies.md` — Password policy, MFA enforcement, session TTL
- `references/auth-webhooks.md` — Auth event webhooks (sign-up, sign-in, deletion)
