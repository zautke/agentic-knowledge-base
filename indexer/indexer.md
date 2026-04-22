---
title: indexer
type: reference
permalink: agent-kb/indexer/indexer
created: 2026-03-17
modified: 2026-03-17
entity_type: reference
status: evergreen
tags:
- project/indexer
- domain/data-pipeline
- domain/search
---

# indexer

Umbrella project for all conversation/code indexing work.

## Sub-projects

| Entity | Path | Status |
|---|---|---|
| **chatgpt-index** | `/Volumes/MACDEV/chatgpt-index/` | Phases 0–7 complete, 51/51 tests passing |
| **assist-indexer** | `/Volumes/MACDEV/assist-indexer/` | Planning hub for chatgpt-index |
| **claude-index** | `/Volumes/FLOUNDER/dev/indexer` | Code search / RAG system with evaluation pipeline |

## Shared Infrastructure

- RAG Evaluation Pipeline (`src/evaluation/`) — shared IR metrics, dual logging, persistent benchmark storage
- See [[RAG Evaluation Pipeline - Module Overview]] and [[RAG Evaluation Metrics Reference]]

## Relations

- child_of [[Fundamental – Agent KB]]
- contains [[chatgpt-index]]
- contains [[assist-indexer]]