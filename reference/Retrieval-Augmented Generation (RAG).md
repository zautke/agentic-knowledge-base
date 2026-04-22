---
title: Retrieval-Augmented Generation (RAG)
type: reference
permalink: kb/reference/retrieval-augmented-generation-rag
entity_type: reference
status: evergreen
created: '2026-03-15'
modified: '2026-03-15'
tags:
- project/agent-kb
- domain/ai
- domain/retrieval
- domain/rag
- status/evergreen
---

# Retrieval-Augmented Generation (RAG)

## Purpose and Scope
Canonical reference note for retrieval-augmented generation as an agent/runtime pattern in the KB.

## Definition
Retrieval-augmented generation combines a retrieval system with a language model so the model can ground its answer in external context fetched at query time instead of relying only on parametric memory.

## Operational Structure
A practical RAG loop has four moving parts:
- retrieval target or corpus
- indexing and storage layer
- retrieval strategy
- generation step that conditions on retrieved evidence

## Implementation Notes
- Dense or hybrid vector retrieval is commonly used to fetch semantically related context.
- Graph retrieval can complement vector retrieval when entity relationships and traversal matter.
- RAG quality depends on indexing quality, retrieval quality, document granularity, and prompt assembly discipline.
- Evaluation should separate retrieval quality from answer quality whenever possible.

## Why It Matters in `agent-kb`
- The KB uses note-level storage plus semantic search and link-graph expansion, which makes retrieval behavior a first-class operational concern.
- Local machine setups may use both vector storage and graph storage to support hybrid retrieval workflows.
- Retrieval notes, indices, and evaluation artifacts should remain linkable as durable graph nodes rather than being buried in session-only summaries.

## Related Evaluation Notes
- [[RAG Evaluation Pipeline - Module Overview]]
- [[RAG Evaluation Metrics Reference]]
- [[Basic Memory Capability Audit and KB Optimization Recommendations]]

## Extended Coverage (2026 Research Sprint)

The following notes extend this reference with deep implementation findings from the March 2026 SOTA research sprint:

### Global Reference (agent-kb/reference/rag/)
- [[Hybrid RAG Architecture – SOTA 2025-2026]] — paradigm shift to context engines, 4 architecture families, RRF fusion, key repos
- [[GraphRAG Plugin System – Factory Pattern]] — entry-point mechanism, 19 plugin categories, PluginBase protocol
- [[Multimodal RAG Ingestion Patterns]] — MIME-type dispatch, Nomic Text/Vision, ColPali, unified vector space
- [[Codebase RAG via AST Patterns]] — tree-sitter v0.25.2 API, AST chunking, code entity/relation schema
- [[RAGAS Evaluation Standard]] — v0.4.3 API, full metric catalog, v0.3→v0.4 breaking changes

### graphrag Project (agent-kb/projects/graphrag/)
- [[graphrag – System Overview]] — production-grade plugin-driven pipeline, 5 stages, 19 ABCs, TUI
- [[graphrag Plugin Categories Registry]] — all 19 plugin ABCs, critical constraints (dimensions(), HybridRetriever gap)
- [[graphrag Pipeline Stage Contracts]] — artifact keys, data flow, retrieval profiles
- [[graphrag SOTA RAG Landscape 2026]] — technology decisions, landscape snapshot, emerging trends
- [[graphrag Plugin Implementation Roadmap]] — 30+ plugins, sequenced build order, minimal viable set

## Relations
- explains [[Agentic Knowledge Base System Overview]]
- related_to [[Fundamental – Agent KB]]
- related_to [[RAG Evaluation Pipeline - Module Overview]]
- related_to [[RAG Evaluation Metrics Reference]]
- related_to [[Local Neo4j and Qdrant Container Operations on largo]]
- extended_by [[Hybrid RAG Architecture – SOTA 2025-2026]]
- extended_by [[graphrag – System Overview]]

## Evolution Log
| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-03-15 | Created canonical RAG reference note. | The KB had RAG evaluation notes but no stable general concept node to anchor retrieval-related links. | User request to add a RAG entity after reviewing KB fundamentals and usage guidance. |

| 2026-03-29 | Added "Extended Coverage" section with forward pointers to all 10 new graphrag-project and reference/rag/ notes created in March 2026 SOTA research sprint | RAG reference was thin; new notes extend it with production implementation details | 3-layer research sprint completion |