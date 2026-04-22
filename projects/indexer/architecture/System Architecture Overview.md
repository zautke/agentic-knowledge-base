---
title: System Architecture Overview
type: reference
permalink: kb/projects/indexer/architecture/system-architecture-overview
tags:
- project/indexer
- domain/architecture
- status/draft
---

# System Architecture Overview

Implementation-state overview for the Repository Context Engine project.

## Current Shape

The project is split across a core TypeScript/Node indexing and retrieval system plus a Next.js UI.

### Core system
- `src/api/*` exposes the public API and orchestration entrypoints
- `src/indexer/*` covers walking, parsing, chunking, embedding, extraction, and related indexing behavior
- `src/retrieval/*` covers retrievers, manager/orchestrator layers, ranking, and retrieval composition
- `src/storage/*` covers SQLite schema/store plus Qdrant integration and related persistence logic
- `src/evaluation/*` covers evaluation harnesses, metrics, storage, and reporting

### UI system
- `ui/src/app/ingestion/*` and `ui/src/app/api/ingestion/*` implement the ingestion UX and API surfaces
- `ui/src/app/architecture-3d/*` and related components implement architecture visualization work
- UI code currently overlaps with ongoing ingestion rewrite work and branch-local logger/infrastructure changes

## Current Active Architectural Themes

- durable ingestion runs with terminal `finished + endStatus` semantics
- SQLite as the durable source of truth for evaluation persistence
- optional Qdrant and Neo4j integration rather than single-backend storage
- coexistence of current implementation docs and more aspirational target-state architecture material

## Important Caveat

`docs/ARCHITECTURE.md` and `docs/UNIFIED_ARCHITECTURE.md` should not both be treated as implementation truth. The former contains legacy sections it explicitly flags; the latter is better treated as target-state synthesis.

## Canonical Companions
- [[Development Operations]]
- [[Repository Documentation Map]]
- [[Documentation Audit and Tech Debt]]
- [[Implementation State - 2026-03-13]]
