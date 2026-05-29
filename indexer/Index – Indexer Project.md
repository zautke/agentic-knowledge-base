---
title: Index – Indexer Project
type: index
permalink: agent-kb/indexer/index-indexer-project
created: 2026-03-17
modified: 2026-03-17
entity_type: index
status: evergreen
tags:
- project/indexer
- meta/index
- source/largo
- machine/largo
---

# Index – Indexer Project

Navigation hub for all indexer sub-projects.

## Root
- [[indexer]] — Umbrella project root

## chatgpt-index (ChatGPT conversation indexer)
- [[chatgpt-index]] — Implementation repo (`/Volumes/MACDEV/chatgpt-index/`)
- [[assist-indexer]] — Planning hub (`/Volumes/MACDEV/assist-indexer/`)
- [[Index – assist-indexer Project]] — Navigation hub for chatgpt-index planning
- [[System Architecture – assist-indexer]] — Settled decisions: halfvec, HNSW, Drizzle, psycopg3
- [[CLAUDE_ARCHITECTURE – chatgpt-index]] — 14 Mermaid diagrams of full implementation

## claude-index (Code search / RAG system)
- Location: `/Volumes/FLOUNDER/dev/indexer`
- [[RAG Evaluation Pipeline - Module Overview]] — Evaluation harness, IR metrics, dual logging
- [[RAG Evaluation Metrics Reference]] — Precision@k, Recall@k, MRR, nDCG formulas

## Session Logs
- [[2026-03-17 Init Session]] — assist-indexer first session: codebase exploration, CLAUDE.md, KB scaffold

## Relations

- child_of [[Index – Agent KB]]

## Trigger Conditions for Update
- A new sub-project is added under the indexer umbrella
- A major implementation milestone is reached in chatgpt-index or claude-index
- New session logs or architecture notes are created for either sub-project

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-03-17 | Created hub with chatgpt-index and claude-index sections | Correct taxonomy: both are entities under project/indexer, not separate top-level projects | User taxonomy correction |