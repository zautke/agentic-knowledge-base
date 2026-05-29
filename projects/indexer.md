---
title: indexer
type: reference
entity_type: reference
status: active
created: 2026-03-17
modified: 2026-05-28
permalink: kb/projects/indexer
tags:
- project/indexer
- domain/code-indexing
- status/active
- stack/typescript
- stack/nextjs
---

# Repository Context Engine (indexer)

> **Note structure:** This is the project **root entity** for `indexer`. The
> navigation hub with the full document map is [[Index – indexer Project]]
> (under `projects/indexer/`). This root note and the `indexer/` folder are
> intentionally paired (folder-note pattern), not duplicates.

Repository Context Engine (`@repo-context/engine`) is a TypeScript code indexing and retrieval system for agentic coding assistants. The repository is currently developed at `/Volumes/FLOUNDER/dev/indexer` and tracks code ingestion, retrieval, evaluation, and deployment concerns across both a core Node/TypeScript library and a Next.js UI.

## Project Overview

- **Type**: software-lib / repository context engine
- **Repository**: `github:/zautke/slurpr`
- **Local Path**: `/Volumes/FLOUNDER/dev/indexer`
- **Primary Package**: `@repo-context/engine`
- **UI Package**: `@repo-context/evaluation-ui`
- **Status**: active development
- **Current Reality**: implementation-state docs, target-state architecture docs, and active rewrite/session notes currently coexist and drift from one another

## Technology Stack

| Layer | Technology |
|---|---|
| Core runtime | TypeScript, Node.js |
| Storage | SQLite, Qdrant, Neo4j |
| Retrieval/indexing | custom indexer + retrieval pipeline |
| UI | Next.js, React, Tailwind, Three.js |
| Testing | Vitest, functional ingestion tests |
| Deploy | Docker, Kubernetes, EKS |

## Active Workstreams

- **Ingestion rewrite**: durable run semantics, stage tracking, status normalization, and UI/API alignment
- **Evaluation persistence**: SQLite-backed evaluation runs, cases, metric scores, and trend/regression support
- **Architecture visualization UI**: 3D/k8s topology work in the Next.js app
- **Infrastructure branch work**: compose, Helm, k8s, server entrypoint, and deployment docs in a segregated worktree
- **Logger branch work**: dedicated ingestion logger UI and port/layout changes in a segregated worktree

## Documentation Map
- [[Index – indexer Project]]
- [[System Architecture Overview]]
- [[RAG Evaluation Pipeline Overview]]
- [[RAG Evaluation Pipeline Design Decisions]]
- [[Development Operations]]
- [[Repository Documentation Map]]
- [[RAG Evaluation Metrics Reference Bridge]]
- [[RAG Evaluation Pipeline API Guide Bridge]]
- [[Documentation Audit and Tech Debt]]
- [[KB Backfill Roadmap]]
- [[Documentation Curation Patterns]]
- [[Implementation State - 2026-03-13]]
- [[Ingestion Status Slice Handoff - 2026-03-05]]
## Known Documentation Debt

1. `README.md`, `docs/ARCHITECTURE.md`, `docs/UNIFIED_ARCHITECTURE.md`, and `docs/INGESTION_OPS.md` describe overlapping but non-identical realities.
2. Canonical `projects/indexer` taxonomy did not exist before 2026-03-17; older project notes live under `indexer/evaluation/*` and `scratch/*`.
3. Two untracked main-branch docs referenced by the audit are still missing from the isolated KB-backfill worktree: `docs/INGESTION_REWRITE_CONTEXT_2026-03-05.md` and `docs/INGESTION_APPROACHES_SYNTHESIS.md`.
4. Semantic retrieval audit remains blocked while `augment-context-engine_codebase-retrieval` returns `401 Unauthorized`.

## Current Operational Notes

- Main branch: `development`
- Active side worktrees: `feat/ingestion-logger`, `feat/docker-k8s-infrastructure`, `feat/indexer-kb-backfill`
- Baseline repo tests in the KB-backfill worktree are blocked by a parent PostCSS config leak from `/Volumes/FLOUNDER/dev/postcss.config.mjs`

## Relations

- child_of [[Fundamental – Agent KB]]
- implements [[Index – indexer Project]]
- related_to [[RAG Evaluation Pipeline - Module Overview]]
- related_to [[RAG Evaluation Pipeline - Design Decisions]]
- related_to [[RAG Evaluation Metrics Reference]]

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-05-28 | Added missing frontmatter fields (entity_type, status, created [from 2026-03-17 canonical-taxonomy date stated in body], modified); added a note-structure clarification distinguishing this root entity from the `[[Index – indexer Project]]` navigation hub and the paired `indexer/` folder. Verified the three pre-canonical RAG-eval relation targets resolve to real notes in the repo-root `indexer/evaluation/` directory (outside the projects/ partition). | Root note missing 4 required frontmatter fields; file-vs-dir naming collision with `projects/indexer/` needed an explicit non-duplicate annotation. | KB curation sweep — projects/ partition. |
