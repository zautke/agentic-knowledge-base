---
title: graphrag SOTA RAG Landscape 2026
type: reference
permalink: agent-kb/projects/graphrag/research/graphrag-sota-rag-landscape-2026
created: 2026-03-29
modified: 2026-03-29
entity_type: reference
status: evergreen
tags:
- project/graphrag
- domain/rag
- domain/research
---

# graphrag SOTA RAG Landscape 2026

Synthesis of 3-layer research sprint (March 2026) mapping SOTA RAG techniques to the graphrag codebase's 19-plugin architecture.

## Key Paradigm Shifts Since 2024

1. **Naive RAG is obsolete** — pure vector similarity retrieval misses structural knowledge in code, entities, and relationships. Hybrid (vector + graph) is now baseline.
2. **RAG → Context Engines** — systems increasingly integrate real-time web, database queries, tool calls alongside retrieved content. Agentic RAG closes the loop.
3. **Multimodal first-class** — PDFs, diagrams, code screenshots require dedicated pipelines, not text OCR fallbacks. ColPali/colqwen2 + Nomic Vision handle documents natively.
4. **Evaluation matured** — RAGAS v0.4 with reference-free metrics makes continuous evaluation at scale practical. No ground-truth corpus needed.

## Architecture Families (mapped to graphrag plugins)

### Family 1: Hybrid RAG (vector + graph)
- Outperforms pure approaches on both retrieval recall and answer faithfulness
- **RRF (Reciprocal Rank Fusion)**: score(d) = Σ 1/(k + rank(d)) — standard fusion; k=60
- Maps to: `hybrid_retriever` + `vector_retriever` + `graph_retriever` + `reranker`
- Key repos: `microsoft/graphrag`, `neo4j/neo4j-graphrag-python`

### Family 2: LightRAG
- Achieves GraphRAG quality at ~50% indexing cost
- Dual-level retrieval: entity-level + relation-level graph traversal
- Incremental graph updates (no full re-index on new docs)
- Maps to: `graph_store` (optimized traversal) + `entity_extractor` (LLM-free where possible)

### Family 3: Agentic RAG
- Retrieval is a tool call in an agent loop, not a fixed pipeline stage
- Query decomposition, iterative refinement, multi-hop graph traversal
- Maps to: `llm_provider` (function calling) + `hybrid_retriever` (tool-wrapped)
- Future evolution of the retrieval + generation stages

### Family 4: Multimodal RAG
- Unified embedding space: text + code + images in same vector index
- MIME-type dispatch at chunker level, modality-specific embedders routed by `Embedder` config
- Maps to: `chunker` (mime-aware dispatch) + `embedder` (Nomic Vision for images, Nomic Text for text/code)

## Technology Landscape Snapshot

### Embedding Models
| Model | Dims | Context | Best For | Cost |
|-------|------|---------|----------|------|
| OpenAI text-embedding-3-small | 1536 | 8191 | General text | $0.02/1M tokens |
| OpenAI text-embedding-3-large | 3072 | 8191 | High-accuracy text | $0.13/1M tokens |
| Nomic Embed Text v1.5 | 768 | 8192 | Local/code/text | Free (Ollama) |
| Nomic Vision v1.5 | 3584 | — | Images aligned to text | Free (Ollama) |
| ColPali/colqwen2 | multi-vec | — | PDF page images | Model download |

### Graph Databases
| Store | Use Case | Query | Local | Production |
|-------|----------|-------|-------|------------|
| NetworkX | Local dev, graph algorithms | Python API | ✅ | ❌ |
| Neo4j v5.28.3 | Production, full async | Cypher | Docker | ✅ |
| FalkorDB | Redis-compatible, fast | Cypher subset | ✅ | ✅ |

### Vector Stores
| Store | Use Case | Dev | Production |
|-------|----------|-----|------------|
| Chroma | Zero-config local | ✅ | ❌ |
| Qdrant | Async-native, filtering | Docker | ✅ |
| pgvector | Postgres-integrated | ✅ | ✅ |

### Entity Extraction
| Method | Zero-shot | Speed | Technical domains |
|--------|-----------|-------|-------------------|
| spaCy en_core_web_trf | ❌ (fixed types) | Fast | Poor |
| GLiNER | ✅ | Medium | Excellent |
| GPT-4 function calling | ✅ | Slow | Excellent |
| tree-sitter (code) | ✅ (deterministic) | Very fast | Code only |

## Key Reference Repositories

| Repo | What to study |
|------|--------------|
| `microsoft/graphrag` | LLM provider/storage factory patterns, community summaries |
| `SciPhi-AI/R2R` | MIME-type ingestion, async pipeline, plugin registration |
| `vitali87/code-graph-rag` | tree-sitter AST → graph schema |
| `neo4j/neo4j-graphrag-python` | GraphStore ABC patterns, Cypher retrieval |
| `stevereiner/flexible-graphrag` | Multi-backend graph store abstraction |

## Emerging Trends (2026)

- **ColPali dominance** for PDF-heavy corpora — avoids lossy OCR, preserves layout
- **LLM-free extraction pipelines** gaining ground (tree-sitter, GLiNER, spaCy) — cost and latency
- **Hybrid chunking** (semantic + AST) for mixed codebases
- **RAGAS continuous eval** replacing end-of-pipeline evaluation
- **Local-first deployment**: Ollama + Chroma + NetworkX for zero-cost dev; swap to cloud for production

## Relations

- Extends → [[reference/retrieval-augmented-generation-rag]]
- Extends → [[reference/rag/hybrid-rag-architecture-sota-2025-2026]]
- Informs → [[projects/graphrag/planning/graphrag Plugin Implementation Roadmap]]
- Related → [[projects/graphrag/research/graphrag Codebase Ingestion via AST]]
- Related → [[projects/graphrag/research/graphrag Multimodal Plugin Patterns]]
- Related → [[projects/graphrag/research/graphrag Evaluation RAGAS Mapping]]

---
*Evolution Log*
- 2026-03-29: Initial synthesis from 3-layer research sprint (Agents A, B, C, D, E)