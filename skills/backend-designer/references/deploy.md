# Template: deploy.md

Always written. Default Docker Compose; add the Coolify section when the user names Coolify or the repo contains Coolify config. Scope with `paths:` to Dockerfile, compose files, CI, and any Coolify files.

```markdown
---
paths:
  - "Dockerfile*"
  - "docker-compose*.yml"
  - "compose*.yml"
  - ".github/**"
  - "coolify*"
---
# Deploy Model

Changelog: ...

## Topology
- Environments: local, staging, production. Same images, different env.
- Services: `app` (web), `worker` (jobs; same image, different command), `postgres`, [`redis`], [`minio` local only], [`<vector engine>`], `proxy` (Traefik/Caddy/nginx; Coolify provides Traefik).

## Image
- Multi-stage Dockerfile: build stage -> slim runtime (`<base>` pinned by digest), non-root user, `HEALTHCHECK` hitting `/health`, no dev deps, no `.env` copied (`.dockerignore` lists `.env*`, `.git`, `node_modules` or `__pycache__`, tests).
- One image for app and worker; only the command differs.

## docker-compose.yml (local)
- `postgres:16` with a named volume, `POSTGRES_*` from `.env`; init SQL enables extensions.
- [`redis:7` with `--appendonly yes`], [`minio/minio` with console port], app and worker with `depends_on` + healthchecks; hot reload via bind mount only in `docker-compose.override.yml`.
- `make up / make migrate / make seed / make logs` (or the npm/just equivalents already in the repo).

## Production
### Docker Compose on a VPS (default)
- `docker-compose.prod.yml`: no bind mounts, `restart: unless-stopped`, resource limits, json logs with rotation, Caddy/Traefik for TLS.
- Deploy = pull image tag -> run migrations as a one-off container -> `up -d --no-deps` rolling restart of `app` then `worker`.

### Coolify (when chosen)
- One Coolify project per environment; app and worker as two resources from the same Dockerfile; Postgres/Redis as Coolify-managed databases with backups enabled to <S3 target>.
- Env vars set in Coolify, not in the repo; build pack: Dockerfile; health check path `/health`; pre-deploy command runs migrations; GitHub/CI webhook triggers deploy on tag.

## CI/CD
- On PR: lint, typecheck, tests against a throwaway Postgres service. On tag or merge to main: build, scan (`trivy`), push `ghcr.io/<org>/<app>:<sha>` and `:latest`, trigger deploy.
- Rollback = redeploy the previous tag; migrations are additive so old code runs on the new schema (database-schema.md).

## Operations
- Backups: Postgres daily dump + WAL (or Coolify scheduled backup) to <bucket>; restore drill <cadence>. Redis AOF and MinIO volumes backed up <how>.
- Logs and metrics: <stack>; uptime check on `/health` from outside.
- Secrets rotation and incident contacts: security.md.
```
