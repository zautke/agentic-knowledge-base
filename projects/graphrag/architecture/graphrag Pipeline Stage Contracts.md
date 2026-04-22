---
title: graphrag Pipeline Stage Contracts
type: reference
permalink: agent-kb/projects/graphrag/architecture/graphrag-pipeline-stage-contracts
created: 2026-03-29
modified: 2026-03-29
entity_type: reference
status: evergreen
tags:
- project/graphrag
- domain/pipeline
- domain/rag
---

# graphrag Pipeline Stage Contracts

The 5 pipeline stages and their input/output artifact contracts. `PipelineContext` is a typed dict that carries artifacts between stages.

## PipelineContext Artifact Keys

```python
ARTIFACT_DOCUMENTS    = "documents"     # list[Document]
ARTIFACT_CHUNKS       = "chunks"        # list[Chunk]
ARTIFACT_EMBEDDINGS   = "embeddings"    # list[Embedding]
ARTIFACT_ENTITIES     = "entities"      # list[Entity]
ARTIFACT_RELATIONS    = "relations"     # list[Relation]
ARTIFACT_HYBRID_RESULT = "hybrid_result"  # HybridResult
ARTIFACT_GENERATOR_RESULT = "generator_result"  # GeneratorResult
```

## Stage 1: Ingestion

**File**: `graphrag/pipeline/stages/ingestion.py`

**Plugin chain**: `loader → preprocessor → chunker → embedder`

**Inputs**: source path(s) from pipeline config

**Outputs**:
- `ARTIFACT_DOCUMENTS`: normalized Document objects (mime_type set, encoding normalized)
- `ARTIFACT_CHUNKS`: Chunk objects (with `document_id` back-reference, `chunk_index`, `start_char`, `end_char`)
- `ARTIFACT_EMBEDDINGS`: Embedding objects (with `chunk_id` back-reference, `vector: list[float]`, `model_name`, `dimensions`)

**Key invariants**:
- Each Chunk has `document_id` set to parent Document's `id`
- Embeddings list is parallel to Chunks list (same length, same order)
- `Document.mime_type` is set by loader or preprocessor — chunker dispatches on it

## Stage 2: Graph Build

**Plugin chain**: `entity_extractor → relation_extractor → graph_store`

**Inputs**: `ARTIFACT_CHUNKS` from Stage 1

**Outputs**:
- `ARTIFACT_ENTITIES`: Entity objects with `label`, `type`, `chunk_ids`, `properties`
- `ARTIFACT_RELATIONS`: Relation objects with `head_entity_id`, `tail_entity_id`, `type`, `properties`

**Key invariants**:
- Entity IDs are stable (same entity mentioned in 2 chunks → same ID, merged)
- Relations reference Entity IDs (not chunk IDs)
- GraphStore persists after stage (not ephemeral)

## Stage 3: Index Build

**Plugin chain**: `vector_store` (upsert)

**Inputs**: `ARTIFACT_EMBEDDINGS` from Stage 1

**Outputs**: None (side effect — embeddings indexed in VectorStore)

**Key invariants**:
- VectorStore must be initialized with `embedder.dimensions()` before upsert
- Chunk IDs stored as metadata in VectorStore for retrieval → chunk mapping

## Stage 4: Retrieval

**Plugin chain**: `vector_retriever → graph_retriever → hybrid_retriever → reranker`

**Inputs**: query string, retrieval config (profile)

**Outputs**:
- `ARTIFACT_HYBRID_RESULT`: `HybridResult` containing `vector_results: list[ScoredChunk]`, `graph_context: Subgraph`, `reranked_results: list[ScoredChunk]`, `fusion_scores: dict[str, float]`

**Key invariants**:
- `ScoredChunk.score` is in [0.0, 1.0] after reranking
- `Subgraph` contains the k-hop neighborhood of entities in retrieved chunks
- HybridRetriever receives both VectorResults and Subgraph (see design gap in registry note)

## Stage 5: Generation

**Plugin chain**: `prompt_builder → llm_provider → generator`

**Inputs**: query string, `ARTIFACT_HYBRID_RESULT` from Stage 4

**Outputs**:
- `ARTIFACT_GENERATOR_RESULT`: `GeneratorResult` with `answer: str`, `citations: list[Citation]`, `confidence: float`, `token_usage: TokenUsage`

**Key invariants**:
- Citations reference Chunk IDs from HybridResult
- `confidence` is a normalized [0.0, 1.0] score (may be model logprob or heuristic)

## Data Model Reference

```python
# graphrag/interfaces/models.py (key fields only)
@dataclass
class Document:
    id: str
    content: str | bytes
    mime_type: str         # "text/plain", "text/x-python", "application/pdf", ...
    source: str            # file path or URL
    metadata: dict

@dataclass
class Chunk:
    id: str
    document_id: str
    content: str
    chunk_index: int
    start_char: int
    end_char: int
    metadata: dict

@dataclass
class Embedding:
    id: str
    chunk_id: str
    vector: list[float]
    model_name: str
    dimensions: int

@dataclass
class Entity:
    id: str
    label: str
    type: str              # "FUNCTION", "CLASS", "PERSON", "ORG", ...
    chunk_ids: list[str]
    properties: dict

@dataclass
class Relation:
    id: str
    head_entity_id: str
    tail_entity_id: str
    type: str              # "CALLS", "IMPORTS", "INHERITS_FROM", ...
    properties: dict
```

## Retrieval Profiles (Stage Lab)

| Profile | Vector | Graph | Fusion |
|---------|--------|-------|--------|
| `dense` | top-20 ANN | — | — |
| `sparse` | BM25 top-20 | — | — |
| `hybrid` | top-10 ANN | — | RRF |
| `graph` | — | 2-hop subgraph | — |
| `full` | top-10 ANN | 2-hop subgraph | RRF + graph boost |

## Relations

- Part of → [[projects/graphrag/architecture/graphrag – System Overview]]
- Implemented by → [[projects/graphrag/planning/graphrag Plugin Implementation Roadmap]]
- Related → [[reference/rag/hybrid-rag-architecture-sota-2025-2026]]

---
*Evolution Log*
- 2026-03-29: Initial creation — all 5 stage contracts documented from codebase inspection