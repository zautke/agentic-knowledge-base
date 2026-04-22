---
title: CLAUDE_ARCHITECTURE – chatgpt-index
type: reference
permalink: agent-kb/indexer/assist-indexer/architecture/claude-architecture-chatgpt-index
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

# CLAUDE_ARCHITECTURE – chatgpt-index

Detailed Mermaid architecture reference for the chatgpt-index implementation.

**Source file:** `/Volumes/MACDEV/assist-indexer/docs/CLAUDE_ARCHITECTURE.md`
**Verified against:** `/Volumes/MACDEV/chatgpt-index/src/` and `embedder/` as of 2026-03-17

---

## Diagrams Contained

| # | Diagram | Type |
|---|---|---|
| 1 | System Context (C4 L1) | C4Context |
| 2 | Docker Service Topology | graph |
| 3 | Ingest Pipeline — detailed flow | flowchart |
| 4 | Asset Resolution — dual URI pattern | flowchart |
| 5 | Content Type Schema — Zod union | graph |
| 6 | Database Schema (ER) | erDiagram |
| 7 | DB Indexes — all 6 indexes | graph |
| 8 | Search Architecture — 3 modes + fallback | flowchart |
| 9 | Python Sidecar — internal architecture | graph |
| 10 | TypeScript Module Dependency Graph | graph |
| 11 | Linearization Algorithm | flowchart |
| 12 | Chunking and Embedding Sequence | sequenceDiagram |
| 13 | Risk Register | quadrantChart |
| 14 | Implementation Status Table | table |

---

## Key Architecture Facts (for quick lookup)

- **Stream parser:** `JSONParser({ paths: ['$.*'], keepStack: false })` — one conversation at a time, 64KB chunks
- **Linearizer:** backwalk `current_node → root`, reverse, BFS fallback, cycle guard via `visited Set`
- **Asset resolver:** `buildFileIndex()` on startup → O(1) map; normalizes both URI schemes; MIME sniff via magic bytes
- **DB client:** `pg Pool` + `registerTypes()` hooked on `pool.on('connect')` — ensures pgvector types before any vector query
- **Vector search:** `SET LOCAL hnsw.iterative_scan = relaxed_order` inside `db.transaction()` for filtered queries
- **Hybrid search:** `FULL OUTER JOIN` of semantic (RANK by `<=>`) + keyword (RANK by `ts_rank_cd`), score = `Σ 1/(60 + rank)`, sidecar-down fallback to fulltext
- **Sidecar client:** `AbortSignal.timeout(30_000)` on all fetch calls; caller catches and degrades gracefully
- **Embedding write path:** `POST /store` → chunk → embed → psycopg3 `pipeline()` mode → `INSERT … ::halfvec`
- **Batch limit:** 2048 texts per `/embed/batch` request (OOM safety from `danny-avila/rag_api`)

---

## Relations

- documents [[chatgpt-index]]
- part_of [[System Architecture – assist-indexer]]
- part_of [[Index – assist-indexer Project]]