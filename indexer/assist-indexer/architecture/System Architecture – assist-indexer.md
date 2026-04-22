---
title: System Architecture – assist-indexer
type: reference
permalink: agent-kb/indexer/assist-indexer/architecture/system-architecture-assist-indexer
created: 2026-03-17
modified: 2026-03-17
entity_type: reference
status: evergreen
tags:
- project/indexer
- domain/architecture
- domain/data-pipeline
- domain/search
---

# System Architecture – assist-indexer

Documents the settled architecture for the ChatGPT conversation indexer. These decisions are **final** — do not re-debate without strong new evidence.

**Source of truth:** `/Volumes/MACDEV/assist-indexer/docs/INITIAL_DESIGN.md`

---

## Full Stack

| Layer | Technology | Notes |
|---|---|---|
| Stream parser | `@streamparser/json-whatwg` | TypeScript-native, zero deps, WHATWG TransformStream |
| Schema validation | Zod v4 `z.discriminatedUnion()` | O(1) type lookup, branded IDs |
| ORM | Drizzle ORM | `halfvec` column type, migrations, `cosineDistance()` |
| HTTP server | Fastify v5 + `fastify-type-provider-zod` | Full JSON Schema required in v5 |
| DB | PostgreSQL 17 + pgvector **0.8.2** | HNSW + GIN indexes |
| Embedder | FastAPI 0.128 + psycopg3 async | Python 3.12-slim + uv |
| Embedding providers | OpenAI `text-embedding-3-small` (1536d) + Ollama `nomic-embed-text` (768d) | Configurable fallback |
| Tokenizer | `tiktoken` `cl100k_base` | 512 tok / 64 overlap chunks |
| Containers | Node 22 Alpine (TS), `python:3.12-slim` (Python) | uv for Python deps |

---

## Settled Architecture Decisions

### halfvec(1536) over vector(1536)
- **Decision:** Use `halfvec(1536)` for all embedding columns
- **Rationale:** 50% storage reduction, 23% faster HNSW builds, equivalent recall
- **Constraint:** Requires pgvector ≥ 0.8.2

### HNSW over IVFFlat
- **Decision:** Use HNSW index on embedding columns
- **Rationale:** No training phase required; better performance on filtered queries; online insertions supported
- **Anti-pattern:** IVFFlat requires rebuild when data distribution shifts

### @streamparser/json-whatwg over stream-json
- **Decision:** `@streamparser/json-whatwg` for parsing 146MB `conversations.json`
- **Rationale:** TypeScript-native, zero dependencies, built on WHATWG TransformStream (Node 22 native)
- **Anti-pattern:** Buffering full 146MB into memory before parse

### Drizzle ORM (only option for this stack)
- **Decision:** Drizzle ORM over raw pg or other ORMs
- **Rationale:** Only TypeScript ORM with native `halfvec` column type and `cosineDistance()` helper
- **Note:** Zod v4 integration requires `drizzle-zod` ≥ compatible version

### psycopg3 async over asyncpg (Python sidecar)
- **Decision:** `psycopg3` async driver in the Python embedding sidecar
- **Rationale:** 152K QPS vs 118K QPS in benchmarks (28% higher); pipeline mode support
- **Date confirmed:** 2026-03-10 (INITIAL_DESIGN.md Phase 1 research)

### python:3.12-slim + uv over Alpine
- **Decision:** `python:3.12-slim` base image, `uv` package manager
- **Rationale:** Alpine breaks pip wheels for numpy/tiktoken due to musl libc; uv is faster than pip
- **Anti-pattern:** Alpine Python for any ML/numerical dependency

### pgvector 0.8.2 (pinned)
- **Decision:** Pin Docker image to `pgvector/pgvector:0.8.2-pg17`
- **Rationale:** CVE-2026-3172 buffer overflow affects earlier versions
- **This is a security constraint, not an optimization**

---

## Content Type Schema

Zod `discriminatedUnion` covering all confirmed content types from the December 2025 export:

```
text | multimodal_text | code | execution_output |
tether_browsing_display | tether_quote | sonic_webpage |
reasoning_recap | thoughts | system_error | app_pairing_content
+ catch-all via z.discriminatedUnion().or(schema)
```

---

## Asset Naming — Both Patterns Present Simultaneously

```
Old:  file-service://files/file-{base58ID}-{descriptor}.{ext}
New:  sediment://file/file_{33hexchars}-sanitized.{ext}
```

**Critical:** Always resolve via MIME sniff fallback. Neither pattern is exhaustive. Both appear in the same export.

---

## Zod v4 Gotchas (confirmed in this project)

| Issue | Wrong | Correct |
|---|---|---|
| Record with value type | `z.record(valueSchema)` | `z.record(z.string(), valueSchema)` |
| UUID version field | `[0-9]` | `[1-8]` only |
| Catch-all union | `.catch()` | `z.discriminatedUnion().or(schema)` |
| Branded IDs | `z.string().brand('X')` | `z.string().brand<'X'>()` |

---

## Data Flow

```
conversations.json (146MB)
    │
    ▼ @streamparser/json-whatwg (streaming — one conversation at a time)
    │
    ▼ Zod discriminatedUnion validation (11+ content types + catch-all)
    │
    ├─► linearize.ts: mapping tree → LinearizedMessage[] (BFS from root)
    │
    ├─► asset-resolver.ts: dual naming pattern + MIME sniff
    │
    ├─► content-extract.ts: text extraction, strip \ue200-\ue204 PUA chars
    │
    ▼ PostgreSQL 17
    ├── conversations (uuid, title, metadata)
    ├── messages (uuid, position, depth, content_text, content_json)
    ├── assets (id, local_path, status: resolved|missing|placeholder)
    ├── search_documents (document_text, tsv: tsvector GENERATED)
    └── embeddings (halfvec(1536), HNSW index)
         │
         ▼ Python sidecar (FastAPI + psycopg3)
         └── tiktoken chunking → OpenAI/Ollama embed → halfvec INSERT
    │
    ▼ Fastify v5 REST API
    ├── GET  /conversations (paginated)
    ├── GET  /conversations/:id/messages
    ├── GET  /conversations/:id/tree
    ├── GET  /assets/:id/file
    ├── POST /search {mode: fulltext|semantic|hybrid}
    ├── POST /ingest
    └── GET  /health, /stats
```

---

## Relations

- documents [[assist-indexer]]
- documents [[chatgpt-index]]
- part_of [[Index – assist-indexer Project]]
- extended_by [[CLAUDE_ARCHITECTURE – chatgpt-index]]