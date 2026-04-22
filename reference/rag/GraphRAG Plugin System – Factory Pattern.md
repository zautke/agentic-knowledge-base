---
title: GraphRAG Plugin System – Factory Pattern
type: reference
permalink: agent-kb/reference/rag/graph-rag-plugin-system-factory-pattern
created: 2026-03-29
modified: 2026-03-29
entity_type: reference
status: evergreen
tags:
- project/agent-kb
- domain/rag
- domain/plugin-architecture
- domain/software-design
- status/evergreen
---

# GraphRAG Plugin System – Factory Pattern

Global reference for the entry-point registry / factory pattern used to build plugin-driven RAG systems in Python. This pattern appears in Microsoft GraphRAG, R2R, and the local graphrag project.

## Core Mechanism: setuptools Entry Points

Plugins register themselves at install-time via `pyproject.toml`:

```toml
[project.entry-points."graphrag.plugins"]
my_chunker = "mypackage.chunkers.semantic:SemanticChunker"
my_embedder = "mypackage.embedders.nomic:NomicEmbedder"
```

At runtime, the registry loads all entry points for the group:
```python
from importlib.metadata import entry_points
eps = entry_points(group="graphrag.plugins")
for ep in eps:
    cls = ep.load()   # Dynamically import the class
    registry.register(cls.plugin_category, ep.name, cls)
```

**Why this matters:** Third-party plugins can be installed as separate packages (`pip install graphrag-plugin-openai`) with zero changes to the core codebase. This is the same mechanism pytest uses for plugins.

## Plugin Protocol (PluginBase ABC)

Every plugin must implement:

```python
class MyPlugin(SomeCategoryABC):
    plugin_category = "chunker"          # Registry lookup key
    
    @property
    def name(self) -> str: ...            # Entry-point key in pyproject.toml
    @property
    def version(self) -> str: ...
    
    @classmethod
    def config_schema(cls) -> type[BaseModel]: ...  # Pydantic config model CLASS
    
    async def initialize(self, config: BaseModel) -> None: ...  # Async setup
    async def health_check(self) -> HealthStatus: ...
    async def teardown(self) -> None: ...           # Resource cleanup
```

**Key rule:** `config_schema()` returns a Pydantic model **class** (not instance). The registry validates config dicts against this schema before calling `initialize()`.

## Config-Driven Instantiation

```yaml
# config.yaml selects plugins by name
ingestion:
  chunker:
    plugin: "semantic_chunker"
    config:
      chunk_size: 512
      overlap: 64
```

```python
# Registry resolves name → class → validates config → initializes
instance = await registry.instantiate("chunker", "semantic_chunker", config_dict)
# Internally: cls = lookup(), schema = cls.config_schema(), typed = schema.model_validate(config_dict), ...
```

## Category Hierarchy (19 for graphrag)

| Stage | Category | ABC | Key Method |
|-------|----------|-----|-----------|
| Ingestion | loader | DocumentLoader | `load(source) → AsyncIterator[Document]` |
| Ingestion | preprocessor | DocumentPreprocessor | `preprocess(doc) → Document` |
| Ingestion | chunker | Chunker | `chunk(doc) → list[Chunk]` |
| Ingestion | embedder | Embedder | `embed(chunks) → list[Embedding]` |
| Graph Build | entity_extractor | EntityExtractor | `extract(chunks) → list[Entity]` |
| Graph Build | relation_extractor | RelationExtractor | `extract(chunks, entities) → list[Relation]` |
| Graph Build | graph_store | GraphStore | `upsert_entities/relations`, `traverse()` |
| Index Build | vector_store | VectorStore | `upsert()`, `search()` |
| Retrieval | vector_retriever | VectorRetriever | `retrieve(query, top_k)` |
| Retrieval | graph_retriever | GraphRetriever | `retrieve(query, depth)` |
| Retrieval | hybrid_retriever | HybridRetriever | `retrieve(query, config)` |
| Retrieval | reranker | Reranker | `rerank(query, results)` |
| Generation | llm_provider | LLMProvider | `complete(messages, params)` |
| Generation | prompt_builder | PromptBuilder | `build(context, query)` |
| Generation | generator | Generator | `generate(llm_response, context)` |
| Evaluation | metric | MetricCalculator | `calculate(context)` |
| Evaluation | reporter | EvaluationReporter | `report(eval_report)` |
| Infrastructure | transport | TransportPlugin | `start()`, `broadcast()` |
| Infrastructure | ui | UIPlugin | `start(transport)`, `on_event()` |

## Null Plugin Pattern (Stubs)

Every category ships with a null implementation that:
- Returns empty lists / no-op operations
- Passes `health_check()` immediately
- Enables end-to-end pipeline testing with zero external dependencies
- Serves as the copy-paste template for real implementations

## Critical Implementation Notes

1. **`Embedder.dimensions()` must be static** (return constant without `initialize()` being called). VectorStore reads it during its own initialization.
2. **`config_schema()` returns class, not instance**: `return MyConfig` not `return MyConfig()`
3. **Entry-point names are lookup keys**: key in `pyproject.toml` must exactly match `registry.lookup(category, name)` call.
4. **Async lifecycle**: `initialize()` is called exactly once; `teardown()` must release all resources.
5. **`plugin_category` class variable** must match the category string used in registry calls.

## Relations

- part_of [[Hybrid RAG Architecture – SOTA 2025-2026]]
- implemented_by [[graphrag: Plugin Categories Registry]]
- related_to [[Agentic Knowledge Base System Overview]]

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-03-29 | Initial creation | Layer 2-A research: plugin factory pattern analysis | graphrag plugin implementation sprint |