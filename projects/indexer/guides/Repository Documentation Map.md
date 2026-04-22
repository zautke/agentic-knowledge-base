---
title: Repository Documentation Map
type: reference
permalink: kb/projects/indexer/guides/repository-documentation-map
tags:
- project/indexer
- domain/guides
- status/draft
---

# Repository Documentation Map

Map of the most important documentation surfaces for the `indexer` repository.

## Root / docs
- `README.md` — broad project overview, currently somewhat stale
- `docs/ARCHITECTURE.md` — implementation-oriented but contains flagged legacy sections
- `docs/UNIFIED_ARCHITECTURE.md` — stronger as target-state synthesis than current-state documentation
- `docs/INGESTION_OPS.md` — older ingestion operations/debugging material; currently outdated
- `docs/kubernetes/OPERATIONS_MANUAL.md` — current and high-signal deploy/runbook documentation

## Source-local docs
- `src/api/README.md`
- `src/retrieval/README.md`
- `src/retrieval/IMPLEMENTATION_SUMMARY.md`
- `src/indexer/INDEXER_README.md`
- `src/indexer/QUICKSTART.md`
- `src/indexer/EMBEDDER_README.md`
- `src/indexer/EXTRACTOR_API.md`

## Tests and deploy docs
- `tests/functional/ingestion/README.md`
- `deploy/README.md`
- `deploy/k8s/README.md`
- `deploy/eks/README.md`

## KB-local pre-canonical notes
- `agent-kb/indexer/evaluation/rag-evaluation-pipeline-module-overview`
- `agent-kb/indexer/evaluation/rag-evaluation-pipeline-design-decisions`
- `agent-kb/indexer/evaluation/rag-evaluation-metrics-reference`
- `memory://indexer/evaluation/rag-evaluation-pipeline-api-guide`
- `scratch/rce-ingestion-status-slice-handoff-2026-03-05`

## Audit Context
The authoritative doc-audit staging files currently live in the repo worktree under `docs/indexer-kb-audit/` and should drive canonical KB backfill.

## Related Notes
- [[Documentation Audit and Tech Debt]]
- [[KB Backfill Roadmap]]
