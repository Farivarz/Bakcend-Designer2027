# Template: api-map.md

Scope with `paths:` to routes/controllers/handlers/resolvers plus any shared API schema folder.

```markdown
---
paths:
  - "<routes dir>/**"
  - "<handlers or controllers dir>/**"
---
# API Map

Changelog: ...

## Style
- Protocol: REST / GraphQL / tRPC / gRPC - the one the codebase uses.
- Base path and versioning: `/api/v1` (version in path; breaking change = new version, never silent).
- Response envelope: exact shape, taken from an existing handler or the one being introduced.
- Error shape: `{ error: { code, message, details? } }` with a stable `code` string per error; HTTP status mirrors the class (400 validation, 401 unauthenticated, 403 forbidden, 404, 409 conflict, 422, 429, 500).
- Pagination: cursor-based for lists that grow unbounded; `limit` capped at <N>.
- Idempotency: mutating endpoints called by clients that retry accept `Idempotency-Key`; store in <Redis/DB> for 24h.

## Auth & authorization
- Mechanism: <JWT/session/API key>, where verified (middleware name/file).
- Roles/scopes table: role -> what it may do.
- Rule: authorization is checked in the handler/service against the resource owner, never only at the route level.

## Resource map
One block per resource. Keep each endpoint to one line.

### <Resource>
| Method | Path | Auth | Request | Response | Notes |
|---|---|---|---|---|---|
| GET | /things | user | query: cursor, limit, filter | Thing[] | cached 60s (see cache-redis.md) |
| POST | /things | user | CreateThing | Thing | idempotent |

## Cross-cutting rules
- Input validation: every body/query/param passes through <library> schema before the handler runs; schema lives next to the handler.
- Rate limits per endpoint class (see load-management.md).
- Long work (over ~2s) returns 202 with a job id and is done by the queue.
- Webhooks in: verify signature, respond 200 fast, process async. Webhooks out: signed, retried with backoff, logged.
- Never expose internal ids of other tenants; never return fields the role cannot read.

## Folder layout for new endpoints
Where the route, handler, validation schema, service, and test for a new endpoint go, matching existing code.
```

Build the resource map from the entities and existing routes found in step 1. Keep endpoints that exist, mark ones that should be added or removed, and do not invent resources the domain summary does not mention.
