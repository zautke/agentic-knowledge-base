---
title: RAG Evaluation Pipeline Design Decisions
type: reference
permalink: kb/projects/indexer/architecture/rag-evaluation-pipeline-design-decisions
tags:
- project/indexer
- domain/architecture
- status/draft
- bridge-note
---

# RAG Evaluation Pipeline Design Decisions

Canonical bridge note for the existing evaluation design-decision document.

## Source Of Truth
- Existing note: [[RAG Evaluation Pipeline - Design Decisions]]
- Existing permalink: `agent-kb/indexer/evaluation/rag-evaluation-pipeline-design-decisions`
- Existing file path: `indexer/evaluation/RAG Evaluation Pipeline - Design Decisions.md`

## Why This Note Exists
This note places the evaluation design-decision material inside the canonical `projects/indexer/architecture` subtree while preserving the original pre-canonical note and its backlinks.

## Current Summary
The original design-decision note records decisions around dual logging, JSON-first storage, graded relevance support, fixed metric k-values, callback-based progress reporting, and comparison-first evaluation storage. Those decisions remain useful historical context, but some storage assumptions need revalidation because main-branch work now expands SQLite-backed evaluation persistence.

## Follow-Up
- preserve the original note as the historical design record
- add a future implementation-status companion once evaluation persistence stabilizes
- reconcile JSON-file assumptions against current SQLite evaluation changes

## Related Notes
- [[RAG Evaluation Pipeline - Design Decisions]]
- [[RAG Evaluation Pipeline Overview]]
- [[RAG Evaluation Metrics Reference Bridge]]
- [[RAG Evaluation Pipeline API Guide Bridge]]
