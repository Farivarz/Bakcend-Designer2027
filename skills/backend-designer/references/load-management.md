# Template: load-management.md

Always written. For a small app most sections are one or two lines; still write them, because the limits are what later code is checked against.

```markdown
# Load Management

Changelog: ...

## Expected load (from evidence or stated assumption)
- Users/day, requests/s at peak, largest payloads, heaviest endpoints. Mark assumptions.

## Limits (enforced in code or proxy)
| Scope | Limit | Where enforced | On exceed |
|---|---|---|---|
| Anonymous, per IP | 60 req/min | <middleware/proxy> | 429 + Retry-After |
| Authenticated, per user | 600 req/min | ... | 429 |
| Login / reset / signup | 5 per 15 min per IP+account | ... | 429, constant time |
| Request body | 1 MB JSON; uploads per object-storage.md | proxy + app | 413 |
| Request timeout | 30s at proxy; handler timeout 25s | ... | 504 |
| DB pool per instance | <N> | ORM config | queue then fail fast after 5s |

## Timeouts & retries (outbound)
- DB statement timeout, Redis command timeout, HTTP calls to third parties: connect 2s / total 10s max, retries at most 2 with jittered backoff, only on idempotent operations. Circuit breaker on any third party the app cannot live without.

## Background work
- Anything over ~2s or depending on a third party runs as a job (cache-redis.md queue, or a DB-backed queue when Redis is not needed: table `jobs` polled with `SKIP LOCKED`).
- Scheduled jobs: list with cadence and the lock that prevents double runs across instances.

## Scaling model
- Stateless app containers; scale horizontally behind <proxy>. Sticky sessions: none (state in DB/Redis).
- Postgres: single primary; add read replicas when read-heavy endpoints exceed <threshold>; replicas used only by endpoints listed here.
- Workers scale independently of web; queue depth is the scaling signal.

## Graceful behavior
- SIGTERM => stop accepting, finish in-flight (25s max), close pools. Readiness endpoint flips before shutdown.
- Degraded modes: cache down, storage down, third party down - each names the user-visible behavior (see component files).

## Observability
- Metrics: request rate/latency/error by route, DB pool usage, queue depth, job failures. Tracing with request id across web -> worker -> DB.
- SLO targets: p95 latency <ms>, availability <%>; alerts in security.md.
```
