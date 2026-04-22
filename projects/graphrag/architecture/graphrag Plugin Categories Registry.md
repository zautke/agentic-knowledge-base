---
title: graphrag Plugin Categories Registry
type: reference
permalink: agent-kb/projects/graphrag/architecture/graphrag-plugin-categories-registry
created: 2026-03-29
modified: 2026-03-29
entity_type: reference
status: evergreen
tags:
- project/graphrag
- domain/plugin-architecture
- domain/rag
---

# graphrag Plugin Categories Registry

All 19 plugin ABCs defined in `graphrag/interfaces/`. Each maps to one entry-point group key in `pyproject.toml`.

## The 19 Plugin Categories

| # | Category Key | ABC Class | Null Plugin | Primary Method Signature |
|---|-------------|-----------|-------------|--------------------------|
| 1 | `loader` | `DocumentLoader` | `null_loader.py` | `load(source: str) -> AsyncIterator[Document]` |
| 2 | `preprocessor` | `DocumentPreprocessor` | `null_preprocessor.py` | `process(doc: Document) -> Document` |
| 3 | `chunker` | `DocumentChunker` | `null_chunker.py` | `chunk(doc: Document) -> list[Chunk]` |
| 4 | `embedder` | `Embedder` | `null_embedder.py` | `embed(texts: list[str]) -> list[Embedding]` + `dimensions() -> int` |
| 5 | `entity_extractor` | `EntityExtractor` | `null_entity_extractor.py` | `extract(chunk: Chunk) -> list[Entity]` |
| 6 | `relation_extractor` | `RelationExtractor` | `null_relation_extractor.py` | `extract(entities: list[Entity], chunk: Chunk) -> list[Relation]` |
| 7 | `graph_store` | `GraphStore` | `null_graph_store.py` | `add_entities()`, `add_relations()`, `query_subgraph()` |
| 8 | `vector_store` | `VectorStore` | `null_vector_store.py` | `upsert(embeddings)`, `search(query_vec, top_k)` |
| 9 | `vector_retriever` | `VectorRetriever` | `null_vector_retriever.py` | `retrieve(query: str, config) -> list[ScoredChunk]` |
| 10 | `graph_retriever` | `GraphRetriever` | `null_graph_retriever.py` | `retrieve(query: str, config) -> Subgraph` |
| 11 | `hybrid_retriever` | `HybridRetriever` | `null_hybrid_retriever.py` | `retrieve(query: str, config) -> HybridResult` |
| 12 | `reranker` | `Reranker` | `null_reranker.py` | `rerank(query: str, results: list[ScoredChunk]) -> list[ScoredChunk]` |
| 13 | `prompt_builder` | `PromptBuilder` | `null_prompt_builder.py` | `build(query: str, context: HybridResult) -> str` |
| 14 | `llm_provider` | `LLMProvider` | `null_llm_provider.py` | `complete(prompt: str, **kwargs) -> str` |
| 15 | `generator` | `Generator` | `null_generator.py` | `generate(prompt: str, llm: LLMProvider) -> GeneratorResult` |
| 16 | `metric_calculator` | `MetricCalculator` | `null_metric_calculator.py` | `calculate(sample: EvalSample) -> MetricResult` |
| 17 | `cache` | `CacheBackend` | `null_cache.py` | `get(key)`, `set(key, value, ttl)`, `delete(key)` |
| 18 | `storage` | `StorageBackend` | `null_storage.py` | `read(path)`, `write(path, data)`, `list(prefix)` |
| 19 | `scorer` | `Scorer` | `null_scorer.py` | `score(query: str, chunk: Chunk) -> float` |

## PluginBase Protocol

Every plugin must implement these from `graphrag/interfaces/base.py`:

```python
class PluginBase(ABC):
    plugin_category: ClassVar[str]      # must match entry-point key
    
    @property
    @abstractmethod
    def name(self) -> str: ...          # unique within category
    
    @classmethod
    @abstractmethod
    def config_schema(cls) -> type[BaseModel]: ...  # Pydantic class
    
    async def initialize(self, config: dict) -> None: ...
    async def health_check(self) -> bool: ...
    async def teardown(self) -> None: ...
```

## Entry-Point Registration Pattern

```toml
# pyproject.toml (lines 41-62)
[project.entry-points."graphrag.plugins"]
null_loader = "graphrag.plugins.null_loader:NullLoader"
# ... one line per plugin ...
my_plugin = "my_package.module:MyPluginClass"
```

Third-party plugins pip-install and declare the same entry-point group — no code changes to graphrag core required.

## Critical Implementation Constraints

### Embedder.dimensions() Must Be Static
`VectorStore.initialize()` calls `embedder.dimensions()` to set its index shape. This happens before `embedder.initialize()` is called. **The value must be derivable from config alone, not from a loaded model.**

```python
# CORRECT — hardcoded from config
def dimensions(self) -> int:
    return self._config.dimensions  # set in config_schema

# WRONG — requires loaded model
def dimensions(self) -> int:
    return self._model.embedding_dim  # model not loaded yet
```

### HybridRetriever Context Access (Design Gap)
`HybridRetriever.retrieve(query, config)` needs access to pre-fetched `VectorResults` and `Subgraph` from `PipelineContext`. Current signatures don't expose this. Three resolution options:
- **Option A**: `PipelineBuilder` injects context reference before stage execution
- **Option B**: `_inject_context(ctx: PipelineContext)` method on PluginBase (recommended)
- **Option C**: Task-local state via `contextvars.ContextVar`

Option B is recommended: minimal ABC change, no circular deps.

### Config Validation
`registry.instantiate()` calls `plugin.config_schema()(**config_dict)` for Pydantic validation before `initialize()`. Config errors surface early with field-level messages.

### Health Check Wiring
`health_check()` is polled by the Dashboard SSE broadcaster. Must return quickly (< 100ms). Network calls must be timeout-bounded.

## Relations

- Implements → [[reference/rag/graph-rag-plugin-system-factory-pattern]]
- Part of → [[projects/graphrag/architecture/graphrag – System Overview]]
- Depends on → [[projects/graphrag/architecture/graphrag Pipeline Stage Contracts]]

---
*Evolution Log*
- 2026-03-29: Initial creation — all 19 categories catalogued from interfaces/ directory inspection