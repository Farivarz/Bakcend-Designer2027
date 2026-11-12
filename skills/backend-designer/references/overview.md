# Template: 00-overview.md

Always written. No `paths:` frontmatter (applies everywhere). Target 60-100 lines.

```markdown
# Backend Overview

Changelog: <date> - initial design from codebase profile.

## Stack (as found)
- Language/runtime: ...   Framework: ...   Package manager: ...
- ORM / query layer: ...  Migrations: ... (tool, folder)
- Auth: ... (mechanism, where enforced)
- Tests: ... (runner, where)
- Existing infra: Dockerfile? compose? CI? `.env.example`?

## Domain summary
Three to eight lines: what the app does, who calls the backend (web, mobile, third parties, cron), and the two or three entities everything else hangs off.

## Decision matrix
| Component | Needed | Decision | Why (one line) |
|---|---|---|---|
| Database | yes | PostgreSQL 16 | default; relational data with FKs |
| Cache / Redis | yes/no | ... | concrete use found in <file>, or "none found" |
| Queue / jobs | yes/no | ... | ... |
| Object storage | yes/no | ... | uploads in <file>, or "no file handling" |
| Vector DB | yes/no | pgvector / dedicated | ... |
| Deploy | yes | Docker Compose / Coolify | ... |

## Non-negotiables (apply to every change)
- Every secret comes from environment; nothing secret is committed. `.env.example` lists every variable with a dummy value.
- Every request is validated at the boundary with <the project's validation library>; the DB is never the first validator.
- Every table has `id`, `created_at`, `updated_at`; soft delete only where listed in database-schema.md.
- Every external call (DB, Redis, storage, HTTP) has a timeout and a documented failure behavior.
- Health endpoint `GET /health` returns 200 only when the DB is reachable.

## Environment variables (canonical names)
List each with purpose; the other rule files reuse these names exactly.

## Provisional decisions
Anything decided from a description rather than code, with what evidence would change it.
```

Fill the matrix from evidence in the code. When a component's "Needed" is "no", the skill writes no file for it; the "Why" column is the record.
