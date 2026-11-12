# Template: object-storage.md

Write only when the codebase handles files. Default: S3-compatible storage (MinIO container locally; Cloudflare R2, S3, or Hetzner Object Storage in production) through the S3 client already in the repo or the official SDK.

```markdown
---
paths:
  - "<upload/storage/media dirs>/**"
---
# Object Storage

Changelog: ...

## Provider
- Local/dev: MinIO in compose (deploy.md). Prod: <provider>, region <r>, bucket per environment: `<app>-<env>`.
- Private by default. Public reads only for <asset class>, served via CDN or public bucket policy; everything else via pre-signed URLs.

## Key layout
`<env>/<domain>/<owner id>/<uuid>.<ext>` - never user-supplied filenames in keys; the original name is stored in the DB.

## DB record
Table `files` (database-schema.md): id, owner_id, bucket, key, content_type, size_bytes, sha256, status (`pending|ready|deleted`), created_at. No other row references the file until status is `ready`.

## Upload flow
1. Client requests an upload => API validates type/size against the allowlist, creates a `files` row (`pending`), returns a pre-signed PUT (15 min max, content-type and size constrained).
2. Client PUTs directly to storage (never through the app servers).
3. Client calls confirm => API HEADs the object, verifies size/type (sniff magic bytes for images/PDFs), sets `ready`. Optional: enqueue virus scan or thumbnail job.
Direct multipart through the app is allowed only for <small class> up to <size>.

## Limits & allowlist
| Class | Types | Max size | Processing |
|---|---|---|---|
| avatar | image/png, image/jpeg, image/webp | 5 MB | resize to 512px, strip EXIF |
| document | application/pdf | 25 MB | none |

## Download
- Pre-signed GET (1 h max) issued only after the authorization check in the service layer. `Content-Disposition` set server-side.

## Lifecycle & reliability
- Orphan sweep job: delete `pending` rows and objects older than 24h. Deleting a row deletes the object (soft-delete first, purge after <N> days).
- Versioning <on/off>; lifecycle rule aborts incomplete multipart uploads after 1 day.
- Storage down => uploads return 503 with retry; reads of already-ready files fall back to <CDN cache / error>.
- Backups: provider replication or scheduled sync to a second bucket; the `files` table is the inventory for restore.
```
