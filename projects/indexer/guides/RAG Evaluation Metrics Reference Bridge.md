---
title: RAG Evaluation Metrics Reference Bridge
type: reference
permalink: kb/projects/indexer/guides/rag-evaluation-metrics-reference-bridge
tags:
- project/indexer
- domain/guides
- status/draft
- bridge-note
---

# RAG Evaluation Metrics Reference Bridge

Canonical bridge note for the existing evaluation metrics reference.

## Source Of Truth
- Existing note: [[RAG Evaluation Metrics Reference]]
- Existing permalink: `agent-kb/indexer/evaluation/rag-evaluation-metrics-reference`
- Existing file path: `indexer/evaluation/RAG Evaluation Metrics Reference.md`

## Why This Note Exists
The original metrics reference predates the canonical `projects/indexer` taxonomy. This bridge keeps that note discoverable from the new project subtree without replacing the original document.

## Current Summary
The original note documents Precision@k, Recall@k, MRR, nDCG, Average Precision, Hit Rate, and latency metrics for the `src/evaluation/metrics.ts` implementation. Audit work suggests it remains broadly useful, though it should eventually be paired with a current implementation-status note for how those metrics are used in the evolving evaluation harness and persistence model.

## Related Notes
- [[RAG Evaluation Metrics Reference]]
- [[RAG Evaluation Pipeline Overview]]
- [[RAG Evaluation Pipeline API Guide Bridge]]
