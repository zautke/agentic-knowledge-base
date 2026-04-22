---
title: Index – indexer Project
type: index
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
