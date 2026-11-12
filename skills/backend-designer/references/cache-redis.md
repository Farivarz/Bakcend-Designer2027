# Template: cache-redis.md

Write only when the overview marks cache/Redis as needed. Each use must be concrete; delete any section without a real use in this codebase.

```markdown
---
paths:
  - "<cache/session/queue dirs>/**"
---
# Cache & Redis

Changelog: ...

## Instance
- Redis <version>, single instance with AOF `everysec` (managed service or the `redis` container in deploy.md). Max memory <size>, eviction `allkeys-lru` for cache keys - which means nothing that must not be lost lives in the cache namespace.
- If any use below requires durability (queues, idempotency keys, counters that gate money), use a separate instance or logical DB with `noeviction`.
- Client: <library already in repo>. Connect timeout 2s, command timeout 1s.

## Key naming
`<app>:<env>:<domain>:<id>[:<field>]`, e.g. `shop:prod:user:123:profile`. Every key has a TTL unless listed under "persistent keys".

## Uses
### 1. <Use name, e.g. read-through cache for GET /things>
- What is cached, key pattern, TTL, invalidation trigger (which write invalidates which key). Stale-while-revalidate: yes/no.
- Failure behavior: cache down => serve from DB, log once per minute, never error to the user.

### 2. Sessions / token revocation
- Key, TTL = token lifetime. Failure behavior: Redis down => fail closed for revocation checks or fail open for sessions - decide and state.

### 3. Rate limiting
- Sliding window per <ip/user/key>; limits in load-management.md. Failure behavior: Redis down => allow with a warning log (availability over strictness) unless the endpoint is listed as fail-closed in security.md.

### 4. Job queue
- Library <BullMQ/Sidekiq/Celery/...>; queues: <names>; retry policy; dead-letter handling; idempotent handlers (a job may run twice).

### 5. Pub/sub or realtime
- Channels and payload shape; delivery is at-most-once - anything that must be durable goes through the DB.

## Rules
- Cache only what is read ~10x more than written, or is expensive to compute. Do not cache per-user data without the user id in the key.
- Never cache authorization decisions longer than the token lifetime.
- Cached values are serialized as <JSON/msgpack>; bump a version segment in the key when the shape changes.
- Memory and hit rate are dashboarded; a hit rate below ~70% means the TTL or key design is wrong.
```
