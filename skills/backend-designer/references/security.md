# Template: security.md

Always written, unscoped. Short: it collects the controls the other files reference so they are not repeated.

```markdown
# Security Baseline

Changelog: ...

## Secrets & config
- All secrets via environment (names in 00-overview.md). Rotation procedure: <where/how>. No secrets in logs, URLs, or error messages.

## Authentication
- <Mechanism>. Passwords hashed with argon2id (or bcrypt cost 12+ if the codebase already uses it). Tokens: <lifetime>, refresh rotation, revocation list in <Redis/DB>.
- Sessions bound to a secure, httpOnly, sameSite cookie or a Bearer header - one, not both.

## Authorization
- Resource-level ownership check in the service layer for every read and write of user-owned data.
- Admin actions behind a separate role and audited (table `audit_log`: actor, action, target, at, ip).

## Input & output
- Validate every input at the boundary (api-map.md). Reject unknown fields.
- Parameterized queries only. HTML output escaped; user-supplied URLs validated against an allowlist of schemes.
- File uploads: see object-storage.md (type sniffing, size cap, no execution path).

## Transport & headers
- TLS everywhere (terminated by <proxy>). HSTS, `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY` or CSP `frame-ancestors`, a CSP for any served HTML, CORS allowlist of exact origins.

## Abuse controls
- Rate limits and body size caps per load-management.md. Login and password-reset endpoints have stricter limits and constant-time responses.

## Dependencies & supply chain
- Lockfile committed; `npm audit` / `pip-audit` / equivalent in CI; base images pinned by digest in deploy.md.

## Logging & monitoring
- Structured logs with request id; never log bodies containing credentials or PII.
- Alert on: 5xx rate, auth failure spikes, DB connection exhaustion, queue depth.

## Incident basics
- Who is paged, where runbooks live, how to rotate the DB password and invalidate all sessions.
```
