---
id: ADR-001
title: "Use JWT with short-lived access tokens and refresh token rotation for API authentication"
status: accepted
superseded_by:
date: 2026-03-26
decision_makers: [lead-architect, security-lead, backend-lead]
informed: [frontend-team, devops-team, product-management]
tags: [security, authentication, api-design, stateless]
scope:
  services: [api-gateway, user-service, order-service, inventory-service]
  paths: ["src/api/**", "src/middleware/auth/**", "src/services/auth/**"]
  layers: [api, application]
enforcement:
  fitness_tests: ["tests/arch/adr-001-auth-enforcement.test.ts"]
  archgate_rules: [".archgate/adrs/ADR-001.rules.ts"]
  ci_required: true
review_trigger: "2027-Q1"
---

# ADR-001: Use JWT with short-lived access tokens and refresh token rotation for API authentication

## Context and problem statement

The platform currently uses long-lived session tokens stored in a central Redis cluster. As the system scales toward a microservices architecture, this approach creates a bottleneck: every service must call the auth service to validate each request, adding latency and a single point of failure. We need an authentication strategy that supports stateless service-to-service validation without sacrificing security.

**Decision question:** What authentication mechanism should we use for the API layer that enables stateless validation across all services while meeting our security and compliance requirements?

---

## Decision drivers

- P95 authentication overhead must remain < 10ms per request (current Redis lookup averages 35ms)
- Must support SOC2 Type II audit trail requirements — all auth events must be logged with user, timestamp, and resource
- Team has strong existing JWT knowledge; zero experience with PASETO or Macaroons
- Zero-trust posture requires short-lived credentials — no token valid for more than 15 minutes without revalidation
- Must work with the existing React SPA and three mobile clients without a complete frontend rewrite

---

## Considered options

1. **JWT with short-lived access tokens + refresh token rotation** — Stateless JWTs (15-min TTL) validated locally; opaque refresh tokens (7-day TTL) stored server-side with single-use rotation
2. **Opaque session tokens with Redis** — Current approach: centralized session store, all services call auth service per request
3. **PASETO (Platform-Agnostic Security Tokens)** — More secure JWT alternative with enforced algorithm specification

---

## Decision outcome

**Chosen option: "JWT with short-lived access tokens and refresh token rotation"**, because it eliminates the Redis validation bottleneck (achieving our P95 < 10ms target), works with existing team expertise, and the 15-minute TTL with server-side refresh token invalidation satisfies the zero-trust posture without requiring full session management infrastructure.

### Consequences

**Positive:**
- Authentication overhead drops from ~35ms to < 5ms (local signature verification only)
- Services validate tokens independently — auth service is no longer in the hot path
- Refresh token rotation provides an invalidation mechanism if a token is compromised
- SOC2 audit trail maintained via centralized refresh token events

**Negative:**
- Access tokens cannot be immediately revoked (15-minute window before expiry). Mitigation: maintain a small token blocklist in Redis for emergency revocations only — not for normal validation
- Refresh token rotation complexity requires careful client-side handling to avoid race conditions on concurrent requests. Mitigation: implement a queue on the client for refresh requests

**Neutral:**
- Requires new refresh token store (PostgreSQL table) — small but non-zero operational overhead

---

## Constraints and enforcement

### Invariants — must ALWAYS hold

- `NEVER` validate JWT tokens inside business logic — authentication MUST occur exclusively in `src/middleware/auth/`
- `ALWAYS` use the `createAuthMiddleware()` factory for route-level auth — never hand-roll token validation inline
- `ALWAYS` log auth events (login, logout, token refresh, revocation) to the audit log via `auditLogger.record()`
- `NEVER` store JWT secrets in source code or environment files committed to version control — use the secrets manager
- `ALWAYS` set `algorithm: 'RS256'` explicitly — never use `algorithm: 'none'` or allow algorithm negotiation

### Preconditions — must be true before using this pattern

- Route must be decorated with `@RequiresAuth` or explicitly marked `@Public` — no routes should be auth-ambiguous
- JWT public key must be retrievable from the secrets manager at service startup
- Refresh token store (PostgreSQL `refresh_tokens` table) must be available and migrated

### Postconditions — guaranteed after correct implementation

- Every authenticated request has a verified `req.user` object containing `{ userId, roles, tokenId }`
- Every auth event produces an entry in the `audit_log` table within the same transaction
- Expired or revoked tokens return `401 Unauthorized` with no additional information leaked in the error body

### Dependency boundaries

```
ALLOWED:
  src/middleware/auth/     → src/services/auth/token-validator.ts
  src/middleware/auth/     → external: jsonwebtoken, jwks-rsa
  src/services/auth/       → src/database/repositories/refresh-token.repository.ts
  src/services/auth/       → external: @company/secrets-manager

FORBIDDEN:
  src/api/routes/**        ✗  direct import of jsonwebtoken
  src/services/orders/**   ✗  any import from src/services/auth/
  src/services/inventory/  ✗  any import from src/services/auth/
  any file                 ✗  external: express-jwt (use createAuthMiddleware() instead)
```

---

## Acceptance criteria

- [ ] `tests/arch/adr-001-auth-enforcement.test.ts` passes in CI (dependency boundary checks)
- [ ] P95 authentication middleware latency < 10ms under 1,000 concurrent requests (k6 load test)
- [ ] Zero direct `jsonwebtoken` imports outside `src/middleware/auth/` and `src/services/auth/` (dependency-cruiser)
- [ ] 100% of routes either have `@RequiresAuth` or `@Public` decorator — no ambiguous routes (AST lint rule)
- [ ] Token with `algorithm: 'none'` rejected with 401 (security unit test: `tests/unit/auth/algorithm-none.test.ts`)
- [ ] Revoked refresh token rejected within 1 second of revocation (integration test)
- [ ] Audit log entry created for every login, logout, and token refresh event (integration test)

---

## Validation and revisit triggers

Revisit this ADR if any of the following occur:
- Authentication-related security incident affecting token validity or leakage
- P95 auth latency exceeds 20ms for 5 or more consecutive days in production
- Regulatory requirement changes necessitating shorter token TTL (< 5 minutes) — this would shift the cost/benefit toward opaque sessions
- Team adopts a dedicated identity provider (Auth0, Cognito) that natively handles this concern
- Scheduled review: 2027-Q1

---

## Alternatives analysis

### Option B: Opaque session tokens with Redis

**How it works:** Each login creates a random session ID stored in Redis with a 24-hour TTL. Every API request sends the session ID; the auth middleware performs a Redis lookup to validate and retrieve the session.

**Rejected because:** Redis lookup adds 30–40ms to every request, violating the P95 < 10ms driver. Redis also becomes a single point of failure — a Redis outage takes down auth for all services simultaneously. Horizontal scaling requires Redis Cluster, increasing operational complexity and cost.

```typescript
// REJECTED PATTERN — do not implement:
app.use(async (req, res, next) => {
  const session = await redis.get(`session:${req.headers['x-session-id']}`);
  if (!session) return res.status(401).json({ error: 'Unauthorized' });
  req.user = JSON.parse(session);
  next();
});
```

### Option C: PASETO

**How it works:** PASETO (Platform-Agnostic Security Tokens) is a more secure alternative to JWT that eliminates algorithm confusion attacks by enforcing a specific algorithm per version (`v4.local` uses XChaCha20-Poly1305, `v4.public` uses Ed25519).

**Rejected because:** While PASETO is technically superior for new systems, zero team members have PASETO implementation experience. The 3-month migration timeline and library ecosystem immaturity (significantly fewer audited libraries vs. battle-tested JWT implementations) outweigh the security improvement given that our JWT implementation already mitigates algorithm confusion via explicit `RS256` enforcement. Revisit if the JWT ecosystem shows signs of major vulnerability or team composition changes.

---

## Context for AI agents

When implementing code affected by this ADR:

1. The reference implementation for `createAuthMiddleware()` is in `src/middleware/auth/auth.middleware.ts` — read this before writing any auth-adjacent code
2. Run `npm run test:arch` after any changes to `src/api/`, `src/middleware/`, or `src/services/` to verify dependency boundaries
3. If you need to add a public (unauthenticated) route, add the `@Public` decorator — never simply omit auth middleware
4. The `req.user` type is defined in `src/types/express.d.ts` — do not redeclare it inline
5. Do NOT refactor existing routes to use the new decorator pattern unless explicitly instructed — this creates unnecessary diff noise in PRs

---

## Related decisions

- **Supersedes:** None (first auth decision for this system)
- **Depends on:** ADR-000: PostgreSQL as primary data store (refresh token table)
- **Related:** ADR-005: API rate limiting strategy, ADR-009: Secrets management approach

---

*Example ADR — BHIL AI Architecture Toolkit — [barryhurd.com](https://barryhurd.com)*
