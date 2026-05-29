---
title: chatgpt-index
type: reference
permalink: agent-kb/indexer/chatgpt-index/chatgpt-index
created: 2026-03-17
modified: 2026-03-17
entity_type: reference
status: evergreen
tags:
- project/indexer
- domain/data-pipeline
- domain/search
- source/largo
- machine/largo
---

# chatgpt-index

**Type:** Implementation repository
**Local path:** `/Volumes/MACDEV/chatgpt-index/`
**Planning hub:** `/Volumes/MACDEV/assist-indexer/` → [[assist-indexer]]
**Architecture:** [[System Architecture – assist-indexer]] | [[CLAUDE_ARCHITECTURE – chatgpt-index]]

## What This Is

TypeScript + Python implementation of the ChatGPT conversation indexer. Parses the December 2025 export (146MB, 651 conversations) into a PostgreSQL + pgvector searchable index with a REST API.

## Stack

| Layer | Technology |
|---|---|
| Runtime | Node.js 22, pnpm |
| API | Fastify v5 + fastify-type-provider-zod |
| ORM | Drizzle ORM + drizzle-kit |
| DB | PostgreSQL 17 + pgvector 0.8.2 |
| Validation | Zod v4 |
| Stream parser | @streamparser/json-whatwg |
| Python sidecar | FastAPI 0.128 + psycopg3 async |
| Embeddings | OpenAI text-embedding-3-small (1536d) + Ollama nomic-embed-text (768d fallback) |
| Chunker | tiktoken cl100k_base (512 tok / 64 overlap) |
| Containers | docker compose (postgres + api + embedder) |

## Implementation Status (as of 2026-03-17)

- Phases 0–7: **complete**
- Tests: **51/51 passing**
- Initial commit: done (57 files, 7168 insertions)
- Search integration tests (`tests/search.spec.ts`): not present in test dir — may be planned

## Key Source Files

| File | Role |
|---|---|
| `src/index.ts` | CLI: `pnpm ingest --export-dir <path>` |
| `src/server.ts` | Fastify server entry |
| `src/schema/content-types.ts` | 11 Zod content types + UnknownContent catch-all |
| `src/parser/stream.ts` | AsyncGenerator via JSONParser `paths: ['$.*']` |
| `src/parser/linearize.ts` | current_node backwalk + BFS fallback |
| `src/parser/asset-resolver.ts` | Dual URI scheme + MIME sniff (magic bytes) |
| `src/search/interface.ts` | SearchProvider + HybridSearchProvider RRF |
| `src/search/vector.ts` | halfvec cosine + iterative HNSW scan in transaction |
| `src/embedder/client.ts` | HTTP client to sidecar (30s timeout) |
| `embedder/embedder/routes.py` | `/embed`, `/embed/batch`, `/store`, `/health` |
| `embedder/embedder/chunker.py` | tiktoken 512/64 chunker singleton |

## Relations

- implemented_by [[assist-indexer]]
- documented_by [[System Architecture – assist-indexer]]
- documented_by [[CLAUDE_ARCHITECTURE – chatgpt-index]]