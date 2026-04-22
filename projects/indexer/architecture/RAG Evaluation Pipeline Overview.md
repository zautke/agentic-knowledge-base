---
title: RAG Evaluation Pipeline Overview
type: reference
permalink: kb/projects/indexer/architecture/rag-evaluation-pipeline-overview
tags:
- project/indexer
- domain/architecture
- status/draft
- bridge-note
---

# RAG Evaluation Pipeline Overview

Canonical bridge note for the existing evaluation pipeline overview material.

## Source Of Truth
- Existing note: [[RAG Evaluation Pipeline - Module Overview]]
- Existing permalink: `agent-kb/indexer/evaluation/rag-evaluation-pipeline-module-overview`
- Existing file path: `indexer/evaluation/RAG Evaluation Pipeline - Module Overview.md`

## Why This Note Exists
The canonical `projects/indexer` taxonomy was created after the original evaluation documentation already existed under a non-canonical subtree. This note anchors that material into the project architecture section without deleting the older note or breaking provenance.

## Current Summary
The evaluation module documents the `src/evaluation` subsystem, including metrics, harnesses, logging, storage, and comparison/trend analysis. Audit work for this repo indicates that evaluation persistence is actively shifting toward more durable SQLite-backed storage and should be revalidated against current code before any deeper rewrite of the original note.

## Follow-Up
- preserve the original note as authoritative until a later migration pass
- add a companion bridge for design decisions, metrics reference, and API guide
- reconcile current evaluation storage behavior against active main-branch diffs in `src/evaluation/*` and `src/storage/*`

## Related Notes
- [[RAG Evaluation Pipeline - Module Overview]]
- [[RAG Evaluation Pipeline Design Decisions]]
- [[RAG Evaluation Metrics Reference Bridge]]
- [[RAG Evaluation Pipeline API Guide Bridge]]
- [[Documentation Audit and Tech Debt]]
