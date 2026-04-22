---
title: graphrag – System Overview
type: reference
permalink: agent-kb/projects/graphrag/architecture/graphrag-system-overview
created: 2026-03-29
modified: 2026-03-29
entity_type: reference
status: evergreen
tags:
- project/graphrag
- domain/rag
- domain/plugin-architecture
- domain/pipeline
---

# graphrag – System Overview

Local codebase: `/Volumes/MACDEV/graphrag`
Branch: `feat/tui-stage-lab`
Python: 3.14 (CPython dev build)

## Purpose

Production-grade, **plugin-driven hybrid RAG pipeline**. Every component is an ABC with a null implementation — the skeleton is complete, real plugins are the remaining work.

## Architecture Layers (C4 summary)

```
┌─────────────────────────────────────────┐
│  Terminal UI (Textual)  │  HTTP Client  │
│  GraphRAGStudioApp      │  JSON-RPC     │
├─────────────────────────────────────────┤
│  JSON-RPC 2.0 Transport (HTTP + SSE)    │
│  POST /rpc   GET /rpc/stream            │
├─────────────────────────────────────────┤
│  AsyncEventBus  ←──────  Dashboard      │
│  (pub/sub, middleware chain)            │
├─────────────────────────────────────────┤
│  Pipeline Orchestrator                  │
│  ingestion → graph_build → index_build  │
│           → retrieval → generation      │
├─────────────────────────────────────────┤
│  ComponentRegistry (singleton)          │
│  discover_entrypoints() → instantiate() │
├─────────────────────────────────────────┤
│  Plugin Layer (19 categories, 16 nulls) │
└─────────────────────────────────────────┘
```

## Key Subsystems

### ComponentRegistry
- Singleton in `graphrag/core/registry/registry.py`
- `discover_entrypoints()` reads `graphrag.plugins` entry-point group
- `register(cls)` → validates ABC compliance, stores by `(category, name)`
- `instantiate(category, name, config_dict)` → calls `plugin.initialize(config)`
- Thread-safe (asyncio.Lock on mutation)

### AsyncEventBus
- Background worker drains event queue continuously
- Middleware chain: `LoggingMiddleware` → `DashboardBroadcastMiddleware`
- `bus.request(ComponentCallRequested)` → awaits `ComponentCallCompleted` (component RPC)
- 18+ typed event classes in `graphrag/core/events/types.py`

### InstrumentedStep
- Auto-wraps every plugin call
- Retry with exponential backoff (configurable attempts)
- Circuit breaker (open/half-open/closed states)
- Fallback chain (try next plugin in list on failure)
- OTel tracing span per call
- MetricSet: latency histogram, error counter, call counter

### Pipeline Stages
| Stage | Key Plugins | Output Artifact |
|-------|------------|-----------------|
| `ingestion` | loader, preprocessor, chunker, embedder | ARTIFACT_DOCUMENTS, ARTIFACT_CHUNKS, ARTIFACT_EMBEDDINGS |
| `graph_build` | entity_extractor, relation_extractor, graph_store | ARTIFACT_ENTITIES, ARTIFACT_RELATIONS |
| `index_build` | vector_store | ARTIFACT_EMBEDDINGS (indexed) |
| `retrieval` | vector_retriever, graph_retriever, hybrid_retriever, reranker | ARTIFACT_HYBRID_RESULT |
| `generation` | prompt_builder, llm_provider, generator | ARTIFACT_GENERATOR_RESULT |

### TUI (Stage Lab)
- `GraphRAGStudioApp` in `graphrag/tui/app.py` — 5-tab Textual interface
- `StageLabController` in `graphrag/tui/controller.py`
- `run_full()`: executes all 5 stages sequentially
- `run_stage(stage_name)`: isolated single-stage execution
- `apply_search_profile(profile_name)`: 5 retrieval profiles (dense, sparse, hybrid, graph, full)

## Transport
- JSON-RPC 2.0 over HTTP POST `/rpc`
- Server-Sent Events at GET `/rpc/stream` for async result streaming
- `/.well-known/agent-card.json` for A2A agent discovery
- Handler registry in `graphrag/dashboard/handlers/registry.py`

## Configuration
- `config/default.yaml`: all-null-plugin defaults
- Plugin selected by `category.plugin` key in YAML → looked up in ComponentRegistry
- Pydantic schema per plugin (returned by `plugin.config_schema()`)
- Runtime validation on `instantiate()`

## Critical Files

| File | Role |
|------|------|
| `graphrag/interfaces/` | All 19 ABCs + data models |
| `graphrag/interfaces/models.py` | Document, Chunk, Embedding, Entity, Relation, Subgraph, HybridResult, GeneratorResult |
| `graphrag/plugins/null_*.py` | 16 null implementation templates |
| `graphrag/core/registry/registry.py` | ComponentRegistry singleton |
| `graphrag/core/events/bus.py` | AsyncEventBus |
| `graphrag/pipeline/stages/ingestion.py` | First real stage to implement |
| `pyproject.toml:41-62` | Entry-point declarations |
| `SYSTEM_ARCHITECTURE.md` | C4 diagrams + design decisions |
| `INITIAL_PLANNING.md` | Existing SOTA research |

## Relations

- Implements → [[reference/rag/graph-rag-plugin-system-factory-pattern]]
- Depends on → [[projects/graphrag/architecture/graphrag Plugin Categories Registry]]
- Depends on → [[projects/graphrag/architecture/graphrag Pipeline Stage Contracts]]

---
*Evolution Log*
- 2026-03-29: Initial creation from codebase exploration + Layer 1 research