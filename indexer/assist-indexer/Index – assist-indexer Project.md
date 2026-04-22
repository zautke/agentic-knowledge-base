---
title: Index – assist-indexer Project
type: index
permalink: agent-kb/indexer/assist-indexer/index-assist-indexer-project
created: 2026-03-17
modified: 2026-03-17
entity_type: index
status: evergreen
tags:
- project/indexer
- meta/index
---

# Index – assist-indexer Project

Navigation hub for the `assist-indexer` planning repository and the `chatgpt-index` implementation it describes.

## Project Root
- [[assist-indexer]] — What the repo is, export data details, implementation status

## Architecture
- [[System Architecture – assist-indexer]] — All settled decisions: halfvec, HNSW, Drizzle, psycopg3, streaming parser, Zod v4 gotchas, asset naming patterns
- [[CLAUDE_ARCHITECTURE – chatgpt-index]] — 14 Mermaid diagrams: C4 context, Docker topology, ingest pipeline, asset resolution, DB schema/indexes, search modes, sidecar internals, module deps, linearizer, embedding sequence, risk quadrant
  - Source: `/Volumes/MACDEV/assist-indexer/docs/CLAUDE_ARCHITECTURE.md`

## Implementation Repo
- [[chatgpt-index]] — Implementation at `/Volumes/MACDEV/chatgpt-index/` (phases 0–7 complete, 51/51 tests passing)

## Session Logs
- [[2026-03-17 Init Session]] — First session: codebase exploration, CLAUDE.md creation, KB priming

## Relations

- child_of [[Fundamental – Agent KB]]
- child_of [[Index – Agent KB]]
- indexes [[assist-indexer]]

## Trigger Conditions for Update
- A new session produces decisions, discoveries, or schema changes not captured in architecture note
- The implementation (`chatgpt-index`) reaches a significant milestone and status needs updating
- A new phase of work begins in the planning repo

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-03-17 | Created index with project root, architecture, and init session log | First session in assist-indexer repo; scaffolding KB presence from scratch | /init + /kb-prime session |
| 2026-03-17 | Added CLAUDE_ARCHITECTURE diagram note link, chatgpt-index implementation link to Architecture and Implementation Repo sections | Architecture detail note and implementation project root were created to fill the two unresolved links from the initial observability report | User request to create architecture diagrams and fill "Still Missing" gaps |