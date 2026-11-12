# Template: database-schema.md

Default engine is PostgreSQL. If the codebase already commits to another engine (MySQL, SQLite, Mongo), keep it and adapt the sections; say in the overview why switching is not worth it.

Scope with `paths:` to models/entities/migrations/db folders.

```markdown
---
paths:
  - "<db or models dir>/**"
  - "**/migrations/**"
---
# Database Schema

Changelog: ...

## Engine & access
- PostgreSQL <version>, extensions: `pgcrypto` (uuid), `citext` (case-insensitive email), `pg_trgm` (search), `pgvector` (only if vector-database.md exists).
- Access through <ORM>; raw SQL allowed only in <folder> and must be parameterized.
- Connection pool: <size> per instance, capped so instances x pool stays under Postgres `max_connections`; add PgBouncer (transaction mode) when the total passes ~80.
- Every query has a statement timeout (`statement_timeout = <ms>` on the app role).

## Conventions
- Tables plural snake_case; columns snake_case; PK `id uuid default gen_random_uuid()` (or bigint identity if the codebase already uses ints - do not mix).
- Every table: `created_at timestamptz not null default now()`, `updated_at timestamptz not null default now()` maintained by trigger or ORM hook.
- Soft delete (`deleted_at timestamptz`) only on: <list>. Everything else hard-deletes with the FK `on delete` rules below.
- Money as `numeric(19,4)` or integer minor units - never float. Timestamps always `timestamptz`.
- Enums as Postgres enums only when values are fixed for life; otherwise a lookup table or `text` + check constraint.

## Tables
One block per table; this is the source of truth for migrations.

### users
| column | type | constraints | notes |
|---|---|---|---|
| id | uuid | pk | |
| email | citext | unique not null | |
| password_hash | text | not null | argon2id; never selected by default |
Indexes: `users_email_key` (unique). FKs: none.

### <next table> ...

## Relationships
ER summary as a list: `orders.user_id -> users.id (on delete restrict)`. State cardinality and the delete rule for every FK.

## Indexing rules
- Every FK column gets an index unless the table is tiny and append-only.
- Index what is filtered/sorted in api-map.md list endpoints; partial indexes for `where deleted_at is null`.
- No index without a query that uses it; remove indexes that `pg_stat_user_indexes` shows unused after a release.

## Migrations
- Tool: <tool>. One migration per change, forward-only in production; "down" exists for local dev.
- Additive first: add nullable column -> backfill in batches -> add constraint. Never rename a column in one step; add, dual-write, switch, drop.
- Long locks forbidden: `create index concurrently`, `add constraint ... not valid` then `validate`.

## Security & reliability
- App connects as a role with least privilege (no superuser, no `create` on schema in prod).
- PII columns listed here: <list>; column-level encryption only where <regulation> requires it, otherwise disk encryption plus access logging.
- Backups: daily base + WAL archiving, restore tested <cadence>; retention <N> days.
- Multi-tenancy: <none / tenant_id column on every tenant table + row-level security policy>.
```
