---
title: RAG Evaluation Pipeline API Guide Bridge
type: reference
permalink: kb/projects/indexer/guides/rag-evaluation-pipeline-api-guide-bridge
tags:
- project/indexer
- domain/guides
- status/draft
- bridge-note
---

# RAG Evaluation Pipeline API Guide Bridge

Canonical bridge note for the existing evaluation API guide.

## Source Of Truth
- Existing note: [[rag-evaluation-pipeline-api-guide]]
- Existing permalink: `memory://indexer/evaluation/rag-evaluation-pipeline-api-guide`
- Existing file path: `indexer/evaluation/rag-evaluation-pipeline-api-guide.md`

## Why This Note Exists
The API guide uses an older/nonstandard permalink shape and predates the canonical `projects/indexer` taxonomy. This bridge anchors it under project guides while preserving the original note.

## Current Summary
The original note documents the `src/evaluation` public API: harness creation, storage factories, logger setup, metrics helpers, and benchmark usage patterns. The guide is still valuable, but current evaluation persistence work on main should be checked before treating every storage detail as current.

## Related Notes
- [[rag-evaluation-pipeline-api-guide]]
- [[RAG Evaluation Pipeline Overview]]
- [[RAG Evaluation Metrics Reference Bridge]]
