---
name: backend-designer
description: Read an application's codebase and design its complete backend infrastructure as separate rule files that Claude Code follows during development — API map, PostgreSQL (or other DB) schema, Redis/cache strategy, load management, object storage, vector database, security baseline, and Docker/Coolify deploy model. Use this whenever the user asks to design, plan, architect, or "set up" a backend, database, schema, caching, scaling, storage, deployment, or infrastructure for an app, or asks for backend rules/guidelines for Claude Code, even if they only name one piece (e.g. "design the database" or "add Redis"), because the pieces depend on each other and should be decided together.
---

# Backend Designer

You produce a backend blueprint for an existing application, written as one file per concern under `.claude/rules/backend/`, so every later Claude Code session builds against the same decisions. The output is rules, not essays: each file states decisions, constraints, and the reason behind them in as few lines as will hold.

Defaults: PostgreSQL for the database, Docker Compose for local and single-server deploys, Coolify when the user names it or the repo already contains Coolify config. Add Redis, a vector database, or object storage only when the codebase shows a need; write a one-line "not needed because …" for anything skipped so the decision is recorded and nobody adds it by accident later.

## Workflow

### 1. Profile the codebase (read before you decide)

Spend real effort here; every file you write depends on it. Collect, by reading code rather than guessing:

- **Stack**: language, framework, package manager, ORM/query layer, existing migrations, test setup.
- **Domain entities**: models/types/schemas, their relations, which fields are unique, searched, sorted, or updated hot.
- **Existing endpoints or routes**, auth mechanism, roles/permissions, and any client (web/mobile) that calls them.
- **Signals for optional components**:
  - Redis: sessions, rate limiting, job/queue libraries, pub/sub, leaderboard-style or hot-read data, idempotency keys.
  - Vector DB: embeddings, semantic search, RAG, LLM calls over user content, "similar items".
  - Object storage: file uploads, images, exports, PDFs, user avatars.
  - Load: cron/background jobs, webhooks, long-running requests, expected concurrency hints in README/issues.
- **Existing infra**: Dockerfile, compose files, Coolify/Fly/Render config, `.env.example`, CI workflows, health endpoints.

Use subagents to fan out the search when the repo is large; you need conclusions, not file dumps. If the repo is brand-new or empty, profile the user's description instead and say which decisions are provisional.

Write the profile into `.claude/rules/backend/00-overview.md` using `references/overview.md`. It includes a decision matrix (component → needed? → why) that the other files must agree with.

### 2. Decide each concern, one file each

Read the matching reference file and write the rule file. Skip a file entirely when the overview marks the component "not needed".

| Output file (`.claude/rules/backend/`) | Reference | Write when |
|---|---|---|
| `00-overview.md` | `references/overview.md` | always |
| `api-map.md` | `references/api-map.md` | always |
| `database-schema.md` | `references/database-schema.md` | always |
| `security.md` | `references/security.md` | always |
| `cache-redis.md` | `references/cache-redis.md` | Redis/cache needed |
| `load-management.md` | `references/load-management.md` | always (may be short) |
| `object-storage.md` | `references/object-storage.md` | uploads/files exist |
| `vector-database.md` | `references/vector-database.md` | embeddings/semantic search exist |
| `deploy.md` | `references/deploy.md` | always |

Principles that apply to every file:

- **Decide, don't survey.** Pick one option and give the reason in one line. A rule file that lists three alternatives is not a rule.
- **Match the codebase.** Use the ORM, naming conventions, folder layout, and auth approach already present. Introduce a new library only when the existing one cannot do the job, and say why.
- **Smallest infrastructure that is secure and reliable.** One Postgres instance with pgvector beats Postgres plus a dedicated vector DB until scale forces otherwise. Redis is justified by a concrete use, not by habit.
- **Security is not optional and not a separate phase.** Every file carries its own security and failure-mode constraints (what is validated, what is encrypted, what happens when the component is down).
- **Make it checkable.** Prefer constraints a reviewer can verify in a diff ("every table has `created_at timestamptz not null default now()`") over aspirations ("tables should be well designed").

### 3. Attach as rules

Claude Code auto-loads every `.md` under `.claude/rules/`. Keep each file focused so the always-loaded cost stays low; put a `paths:` frontmatter on files that only matter for part of the tree, for example:

```yaml
---
paths:
  - "src/db/**"
  - "**/migrations/**"
---
```

Scope guide: `database-schema.md` → db/models/migrations; `api-map.md` → routes/controllers/handlers; `object-storage.md` → upload/storage modules; `deploy.md` → Dockerfile, compose, `.github/**`, `coolify*`. Leave `00-overview.md` and `security.md` unscoped; they apply everywhere.

Then add one line to the project's `CLAUDE.md` (create it if missing) under a "Backend design" heading: "Backend architecture decisions live in `.claude/rules/backend/`. Read `00-overview.md` before changing data, API, or infra. Update the matching rule file in the same change when a decision changes." That last sentence is what keeps the rules alive.

### 4. Report

Finish with a short summary for the user: which files were written, which components were skipped and why, and the three to five decisions most worth a second look (anything provisional, anything that changes cost, anything that conflicts with existing code). Do not paste the files into the chat.

## When the user asks for only one piece

"Design the database" still runs step 1 and writes `00-overview.md`, because the schema depends on what is cached, stored, and searched. Write the requested file fully and the others only as entries in the overview's decision matrix, then offer to complete the rest.

## When rules already exist

If `.claude/rules/backend/` is already populated, read it first and treat it as the current decisions. Change a decision only when the codebase contradicts it or the user asks, and record the reason in the file's "Changelog" line. Never silently rewrite a file the user may have hand-edited; show the diff of decisions in your report.
