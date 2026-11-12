# Changelog

All notable changes to this skill are documented here.
This project follows [Semantic Versioning](https://semver.org/).

## [1.0.0] - 2026-11-12

First public release.

### Added
- `backend-designer` skill: profiles a codebase, fills a decision matrix, and writes one
  rule file per backend concern into `.claude/rules/backend/`.
- Nine reference templates: overview, API map, database schema, security, cache and Redis,
  load management, object storage, vector database, deploy.
- PostgreSQL-first defaults, with pgvector preferred over a separate vector engine until
  scale justifies one.
- Docker Compose and Coolify deploy models.
- Path-scoped rule output, so only the relevant rules load for the files being edited.
- Two eval cases with graders: a full Express SaaS design, and a Django schema-only request
  that has to skip the components the codebase does not need.
