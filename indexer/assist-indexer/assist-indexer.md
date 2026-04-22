---
title: assist-indexer
type: reference
permalink: agent-kb/indexer/assist-indexer/assist-indexer
created: 2026-03-17
modified: 2026-03-17
entity_type: reference
status: evergreen
tags:
- project/indexer
- domain/data-pipeline
- domain/search
---

# assist-indexer

**Type:** Planning and research repository
**Local path:** `/Volumes/MACDEV/assist-indexer/`
**Implementation target:** `/Volumes/MACDEV/chatgpt-index/`
**Primary reference:** `docs/INITIAL_DESIGN.md` (2446-line corrected design for December 2025 export)

## What This Is

`assist-indexer` is a **research and planning repository** — not an implementation repo. It documents the full architecture, schema decisions, and technology choices for the ChatGPT conversation indexer. The actual implementation lives (or will live) at `/Volumes/MACDEV/chatgpt-index/`.

The repo contains:
- `docs/INITIAL_DESIGN.md` — the corrected, authoritative design based on actual December 2025 export analysis
- `docs/research/04-export-schema-analysis.md` — confirmed schema from actual export sample
- `docs/research/01-sota-research.md` — technology landscape justifications
- `chatgpt-indexer.md` — original July 2025 plan (superseded)
- `CLAUDE.md` — agent guidance for working in this repo (created 2026-03-17)

## Export Data Being Indexed

- **Source:** `/Users/luke/Library/CloudStorage/GoogleDrive-lukezautke@gmail.com/My Drive/ai/chat_exports/chatgpt_export_12-27-2025/`
- **Size:** 146MB `conversations.json`
- **Scale:** 651 conversations, 14,930 messages, 233+ assets
- **User ID:** `user-fptgpXo0vJfRcNhjPyiBNCQJ`
- **Asset naming:** Two simultaneous patterns — `file-service://files/file-{base58ID}-{descriptor}.{ext}` (old) and `sediment://file/file_{33hexchars}-sanitized.{ext}` (new)

## Target System (chatgpt-index)

See [[System Architecture – assist-indexer]] for settled decisions.

Brief stack summary:
- TypeScript API: Fastify v5, Drizzle ORM, Zod v4, pnpm, Node 22
- Database: PostgreSQL 17 + pgvector 0.8.2 (`halfvec(1536)`, HNSW index)
- Python sidecar: FastAPI + psycopg3 async, OpenAI `text-embedding-3-small` + Ollama fallback

## Implementation Status

As of 2026-03-17: planning complete, INITIAL_DESIGN.md is authoritative. `chatgpt-index` repo at `/Volumes/MACDEV/chatgpt-index/` has 51/51 tests passing with phases 0–7 complete.

## Relations

- planning_for [[chatgpt-index]]
- child_of [[Fundamental – Agent KB]]
- documented_by [[System Architecture – assist-indexer]]