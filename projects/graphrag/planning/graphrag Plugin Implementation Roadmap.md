---
title: graphrag Plugin Implementation Roadmap
type: reference
permalink: agent-kb/projects/graphrag/planning/graphrag-plugin-implementation-roadmap
created: 2026-03-29
modified: 2026-03-29
entity_type: reference
status: evergreen
tags:
- project/graphrag
- domain/planning
- domain/plugin-architecture
---

# graphrag Plugin Implementation Roadmap

Sequenced build order for 30+ plugins, grounded in 3-layer SOTA research (March 2026). Each tier unblocks the next.

## Technology Decisions (from Layer 3 synthesis)

| Dimension | Local/Dev | Production |
|-----------|-----------|------------|
| Embedder | Nomic Embed Text v1.5 (Ollama) | OpenAI text-embedding-3-small |
| Image Embedder | Nomic Vision v1.5 (Ollama) | Nomic Vision v1.5 (API) |
| Vector Store | Chroma (sync→async bridge) | Qdrant (async-native) |
| Graph Store | NetworkX (pickle persistence) | Neo4j v5.28.3 (full async driver) |
| Entity Extraction | GLiNER (zero-shot, technical prose) | GLiNER or GPT-4 function calling |
| Code Entity Extraction | tree-sitter (deterministic) | tree-sitter |
| Chunker (text) | Fixed-size (token-aware) | Semantic (sentence-transformers) |
| Chunker (code) | AST (tree-sitter) | AST (tree-sitter) |
| LLM Provider | Ollama (local) | OpenAI GPT-4o / Anthropic Claude |
| LLM Proxy | LiteLLM (100+ providers, unified SDK) | LiteLLM |
| Evaluation | RAGAS v0.4.3 | RAGAS v0.4.3 |

## Priority 1 — Foundation (unblocks everything else)

These 3 plugins enable the first end-to-end embedding of any document.

| Plugin Name | Category | Library | Notes |
|-------------|----------|---------|-------|
| `openai_embedder` | `embedder` | `openai` | text-embedding-3-small, 1536 dims; production default |
| `nomic_text_embedder` | `embedder` | `nomic` | 768 dims, Matryoshka; local-first |
| `chroma_vector_store` | `vector_store` | `chromadb` | zero-config; async bridge required (Chroma is sync) |

**Key constraint**: `Embedder.dimensions()` must return deterministic value from config alone — set in `config_schema()`.

## Priority 2 — Ingestion (stated first use case)

First end-to-end pipeline run: Priority 1 + Priority 2.

| Plugin Name | Category | Library | Notes |
|-------------|----------|---------|-------|
| `filesystem_loader` | `loader` | stdlib + `aiofiles` | glob patterns, MIME detection via `python-magic` |
| `codebase_loader` | `loader` | `gitpython` | git-aware, respects `.gitignore`, sets `mime_type` by ext |
| `text_preprocessor` | `preprocessor` | `charset-normalizer` | encoding normalization, whitespace cleanup |
| `fixed_chunker` | `chunker` | custom | token-aware (tiktoken), configurable size + overlap |
| `semantic_chunker` | `chunker` | `sentence-transformers` | topic boundary detection, BiEncoder |
| `ast_chunker` | `chunker` | `tree-sitter` + `tree-sitter-python` | function/class-level splits, 170+ langs |
| `mime_dispatch_chunker` | `chunker` | custom | dispatch table → routes to above chunkers by `mime_type` |

**MIME dispatch chunker** wraps the other chunkers — should be the default `chunker` plugin in `config/default.yaml`.

## Priority 3 — Graph Build (GraphRAG differentiator)

Required for graph-augmented retrieval. Build after Priority 2 produces real chunks.

| Plugin Name | Category | Library | Notes |
|-------------|----------|---------|-------|
| `gliner_entity_extractor` | `entity_extractor` | `gliner` | zero-shot; labels from config; best for mixed corpora |
| `ast_entity_extractor` | `entity_extractor` | `tree-sitter` | FUNCTION/CLASS/MODULE from AST metadata in chunk |
| `ast_relation_extractor` | `relation_extractor` | `tree-sitter` | CALLS/IMPORTS/INHERITS_FROM from call graph |
| `llm_relation_extractor` | `relation_extractor` | `openai` | GPT-4 function calling for prose documents |
| `networkx_graph_store` | `graph_store` | `networkx` | local dev; pickle persistence; NetworkX API |
| `neo4j_graph_store` | `graph_store` | `neo4j` v5.28.3 | production; full async driver; Cypher queries |

## Priority 4 — Retrieval (value delivery)

Enables end-to-end query → answer loop.

| Plugin Name | Category | Library | Notes |
|-------------|----------|---------|-------|
| `dense_vector_retriever` | `vector_retriever` | `chromadb` / `qdrant` | ANN search; top-k config |
| `bm25_vector_retriever` | `vector_retriever` | `rank_bm25` | sparse retrieval for keyword queries |
| `networkx_graph_retriever` | `graph_retriever` | `networkx` | k-hop subgraph traversal, entity lookup |
| `neo4j_graph_retriever` | `graph_retriever` | `neo4j` | Cypher-based; configurable hop depth |
| `rrf_hybrid_retriever` | `hybrid_retriever` | custom | Reciprocal Rank Fusion; k=60; needs context injection (Option B) |
| `cross_encoder_reranker` | `reranker` | `sentence-transformers` | `ms-marco-MiniLM-L-6-v2`; top-20 → top-5 |

**Design fix needed**: `HybridRetriever` requires `_inject_context(ctx: PipelineContext)` on `PluginBase` to access pre-fetched VectorResults and Subgraph. Implement Option B before building `rrf_hybrid_retriever`.

## Priority 5 — Generation

Full end-to-end RAG pipeline.

| Plugin Name | Category | Library | Notes |
|-------------|----------|---------|-------|
| `openai_llm_provider` | `llm_provider` | `openai` | GPT-4o; streaming support |
| `anthropic_llm_provider` | `llm_provider` | `anthropic` | Claude 3.5 Sonnet / Claude 4; streaming |
| `ollama_llm_provider` | `llm_provider` | `ollama` | local; Llama 3.1, Mistral |
| `litellm_llm_provider` | `llm_provider` | `litellm` | 100+ provider proxy; unified config |
| `citation_prompt_builder` | `prompt_builder` | custom | citation markers, context assembly, token budget |
| `citation_generator` | `generator` | custom | source attribution, dedup, confidence scoring |

## Priority 6 — Evaluation

Continuous quality measurement.

| Plugin Name | Category | Library | Notes |
|-------------|----------|---------|-------|
| `ragas_faithfulness` | `metric_calculator` | `ragas` v0.4.3 | `single_turn_ascore()`; gpt-4o-mini sufficient |
| `ragas_answer_relevancy` | `metric_calculator` | `ragas` | reference-free |
| `ragas_context_precision` | `metric_calculator` | `ragas` | retrieval quality |
| `ragas_context_recall` | `metric_calculator` | `ragas` | needs reference; use sparingly |

## Minimal Viable Plugin Set (first working pipeline)

To replace ALL null plugins and demonstrate end-to-end retrieval:
1. `filesystem_loader`
2. `text_preprocessor`
3. `fixed_chunker`
4. `nomic_text_embedder`
5. `chroma_vector_store`
6. `gliner_entity_extractor`
7. `ast_relation_extractor` (or `llm_relation_extractor`)
8. `networkx_graph_store`
9. `dense_vector_retriever`
10. `networkx_graph_retriever`
11. `rrf_hybrid_retriever` (after Option B context injection fix)
12. `cross_encoder_reranker`
13. `citation_prompt_builder`
14. `openai_llm_provider`
15. `citation_generator`

That's 15 plugins across 5 stages — a complete, running pipeline.

## pyproject.toml Registration

Every new plugin must be added to `[project.entry-points."graphrag.plugins"]` in `pyproject.toml` (lines 41-62 pattern), then `pip install -e .` to re-register entry points.

## Relations

- Depends on → [[projects/graphrag/architecture/graphrag Pipeline Stage Contracts]]
- Depends on → [[projects/graphrag/architecture/graphrag Plugin Categories Registry]]
- Informed by → [[projects/graphrag/research/graphrag SOTA RAG Landscape 2026]]
- Informed by → [[projects/graphrag/research/graphrag Codebase Ingestion via AST]]
- Informed by → [[projects/graphrag/research/graphrag Multimodal Plugin Patterns]]
- Informed by → [[projects/graphrag/research/graphrag Evaluation RAGAS Mapping]]

---
*Evolution Log*
- 2026-03-29: Initial roadmap from 3-layer research sprint; technology decisions locked for local-first strategy