# Template: vector-database.md

Write only when embeddings, semantic search, RAG, or "similar items" exist in the code or the user's description. Default: `pgvector` inside the existing Postgres. Move to a dedicated engine (Qdrant, Weaviate, Milvus, Pinecone) only when the overview records a reason: more than ~5-10M vectors, strict sub-50ms p99 at high QPS, or heavy metadata filtering that pgvector's HNSW handles poorly.

```markdown
---
paths:
  - "<search/embeddings/ai dirs>/**"
---
# Vector Database

Changelog: ...

## Engine
- pgvector <version> in the main Postgres (extension enabled in migration <n>). Or dedicated: <engine>, deployed per deploy.md.
- Embedding model: <provider/model>, dimensions <d>, cost per 1k tokens. Model name and dimension are stored with every vector so a model change can be migrated.

## Tables / collections
### <entity>_embeddings
| column | type | notes |
|---|---|---|
| id | uuid pk | |
| <entity>_id | uuid fk on delete cascade | |
| chunk_index | int | 0 for whole-document embeddings |
| content | text | the exact text embedded (needed for re-embedding and display) |
| embedding | vector(<d>) | |
| model | text | |
| metadata | jsonb | tenant_id, language, tags used for filtering |
| created_at | timestamptz | |
Index: `hnsw (embedding vector_cosine_ops) with (m = 16, ef_construction = 64)`; btree on `(<entity>_id)`; btree on filter columns pulled out of metadata when filtered often.

## Chunking & ingestion
- Chunk size <tokens> with <overlap>; chunker lives in <module>. Ingestion runs as a job (load-management.md) triggered on create/update; a content hash skips re-embedding unchanged text.
- Deleting the source row cascades; updating it re-embeds only changed chunks.

## Query
- Top-k = <k> with cosine distance; `SET hnsw.ef_search = <n>` per query for recall. Always filter by tenant/owner before or with the vector search; never post-filter across tenants.
- Hybrid: combine with `pg_trgm` or full-text score via reciprocal rank fusion when exact terms matter (ids, names).
- Timeout <ms>; degraded mode when the embedding provider is down: fall back to keyword search and flag the response.

## Security
- The embedding provider receives only content the owning user is allowed to see; PII is redacted before embedding where <policy> requires.
- Vectors are treated as derived PII: deleted with the source, included in data-export and deletion requests.

## Capacity
- Expected vectors: <n>; at <d> dims about <GB> plus index. Re-evaluate the engine when past <threshold>.
```
