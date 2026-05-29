---
title: Index – indexer Project
type: index
entity_type: index
status: evergreen
created: 2026-03-17
modified: 2026-05-28
permalink: kb/projects/indexer/index-indexer-project
tags:
- project/indexer
- meta/index
- status/evergreen
---

# Index – indexer Project

Navigation hub for the Repository Context Engine project knowledge.

## Project Root
- [[indexer]]

## Architecture
- [[System Architecture Overview]]
- [[RAG Evaluation Pipeline Overview]]
- [[RAG Evaluation Pipeline Design Decisions]]

## Operations
- [[Development Operations]]

## Guides
- [[Repository Documentation Map]]
- [[RAG Evaluation Metrics Reference Bridge]]
- [[RAG Evaluation Pipeline API Guide Bridge]]

## Analysis
- [[Documentation Audit and Tech Debt]]

## Planning
- [[KB Backfill Roadmap]]

## Patterns
- [[Documentation Curation Patterns]]

## Sessions
- [[Implementation State - 2026-03-13]]
- [[Ingestion Status Slice Handoff - 2026-03-05]]
- [[Ingestion Logger Worktree Status - 2026-03-13]]
- [[Docker K8s Infrastructure Worktree Status - 2026-03-13]]

## Existing Pre-Canonical Material To Bridge
- [[RAG Evaluation Pipeline - Module Overview]]
- [[RAG Evaluation Pipeline - Design Decisions]]
- [[RAG Evaluation Metrics Reference]]
- [[rag-evaluation-pipeline-api-guide]]
- [[rce-ingestion-status-slice-handoff-2026-03-05]]

## Related Domains (Global)
- [[Fundamental – Agent KB]]
- [[Index – Agent KB]]

## Current Audit Constraints
- Semantic repository retrieval is still blocked in-session because `augment-context-engine_codebase-retrieval` is returning `401 Unauthorized`.
- The canonical project taxonomy was created after older `indexer/evaluation/*` notes already existed, so migration must preserve provenance and backlinks.
- The "Existing Pre-Canonical Material To Bridge" links above resolve to notes in the **repo-root `indexer/` directory** (outside the `projects/` partition). Reconciling the root-level `indexer/` tree with this canonical `projects/indexer/` tree is deferred to a cross-partition index reconciliation phase.

## Relations

- part_of [[Fundamental – Agent KB]]
- indexes [[indexer]]
- related_to [[RAG Evaluation Pipeline - Module Overview]]
- related_to [[RAG Evaluation Pipeline - Design Decisions]]
- related_to [[RAG Evaluation Metrics Reference]]

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-05-28 | Added missing frontmatter fields (entity_type, status, created, modified); added a Relations block and an audit-constraint note flagging that the pre-canonical bridge targets live in the repo-root `indexer/` tree (cross-partition reconciliation deferred). | Hub missing 4 required frontmatter fields and had no Relations block / Evolution Log. | KB curation sweep — projects/ partition. |
