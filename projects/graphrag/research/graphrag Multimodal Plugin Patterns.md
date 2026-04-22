---
title: graphrag Multimodal Plugin Patterns
type: reference
permalink: agent-kb/projects/graphrag/research/graphrag-multimodal-plugin-patterns
created: 2026-03-29
modified: 2026-03-29
entity_type: reference
status: evergreen
tags:
- project/graphrag
- domain/multimodal
- domain/rag
- domain/research
---

# graphrag Multimodal Plugin Patterns

How to implement multimodal ingestion (text, code, images, PDFs) in the graphrag plugin architecture using MIME-type dispatch and modality-aligned embedding spaces.

## Core Pattern: MIME-Type Dispatch Chunker

The `DocumentChunker` ABC receives a `Document` with `mime_type` set. A dispatch chunker routes to specialized sub-chunkers:

```python
class MIMEDispatchChunker(DocumentChunker):
    plugin_category = "chunker"

    DISPATCH_TABLE = {
        "text/plain":             TextChunker,
        "text/markdown":          TextChunker,
        "text/x-python":          ASTChunker,
        "application/javascript": ASTChunker,
        "application/pdf":        ColPaliChunker,   # page-image extraction
        "image/png":              ImagePassthroughChunker,
        "image/jpeg":             ImagePassthroughChunker,
    }

    async def chunk(self, doc: Document) -> list[Chunk]:
        chunker_cls = self.DISPATCH_TABLE.get(
            doc.mime_type,
            TextChunker  # fallback
        )
        return await chunker_cls(self._config).chunk(doc)
```

Key: `Document.mime_type` is set by `codebase_loader` or `filesystem_loader` based on file extension or magic bytes. The chunker never has to sniff content.

## Embedding Model Strategy

### For Text and Code

**Nomic Embed Text v1.5** (local-first):
```python
# pip install nomic
from nomic import embed
import asyncio

class NomicTextEmbedder(Embedder):
    plugin_category = "embedder"
    
    def dimensions(self) -> int:
        return self._config.dimensions  # 768 (full) or 256/512 (Matryoshka)
    
    async def embed(self, texts: list[str]) -> list[Embedding]:
        # Prefix for task type — required for Nomic v1.5
        prefixed = [f"search_document: {t}" for t in texts]
        
        # Run sync SDK in executor
        loop = asyncio.get_event_loop()
        output = await loop.run_in_executor(
            None,
            lambda: embed.text(
                prefixed,
                model="nomic-embed-text-v1.5",
                task_type="search_document",
            )
        )
        embeddings = output["embeddings"]
        return [
            Embedding(
                id=f"emb_{i}",
                chunk_id=...,  # set by caller
                vector=embeddings[i],
                model_name="nomic-embed-text-v1.5",
                dimensions=len(embeddings[i]),
            )
            for i in range(len(texts))
        ]
```

**Task prefix strings** (critical for quality):
| Use case | Prefix |
|----------|--------|
| Indexing documents | `search_document:` |
| Querying | `search_query:` |
| Classification | `classification:` |
| Clustering | `clustering:` |

### For Images (PDFs, diagrams, screenshots)

**Nomic Vision v1.5** (3584 dims, aligned with Text v1.5):
```python
from nomic import embed

# Images are passed as PIL Images or file paths
output = embed.image(
    images=pil_image_list,
    model="nomic-embed-vision-v1.5",
)
# Returns same 3584-dim space as Nomic Text — direct cross-modal ANN
```

**ColPali / colqwen2** (multi-vector, late interaction):
- Better than single-vector for PDF page retrieval
- Each page → 256 vectors × 128 dims (token-level representations)
- Query → 30-50 vectors; score = MaxSim(query_vecs, doc_vecs)
- Use when precision on document layout matters more than ANN speed
- Storage: ~256KB/page (vs 3584 floats × 4 bytes = ~14KB for Nomic Vision)

```python
# pip install colpali-engine
from colpali_engine.models import ColQwen2, ColQwen2Processor

model = ColQwen2.from_pretrained("vidore/colqwen2-v0.1")
processor = ColQwen2Processor.from_pretrained("vidore/colqwen2-v0.1")

# Index: process PDF pages as images
page_embeddings = model(**processor.process_images(pages))

# Query: late interaction scoring
query_embeddings = model(**processor.process_queries(["what is X?"]))
scores = processor.score_multi_vector(query_embeddings, page_embeddings)
```

## Unified vs. Separate Vector Spaces

| Strategy | How | When |
|----------|-----|------|
| **Unified** | One VectorStore, all modalities | Nomic Text + Vision (pre-aligned) |
| **Separate** | One VectorStore per modality | Different model families |
| **Hybrid** | Text+code unified, images separate | Most production systems |

For graphrag: **Text + code in Nomic Text space (768 dims), images in Nomic Vision space (3584 dims)** — simplest path to cross-modal search while keeping index sizes manageable.

## PDF Handling Strategy

| Approach | Quality | Complexity |
|----------|---------|------------|
| pdfplumber text extraction + TextChunker | Low (misses layout) | Low |
| pdfplumber + image extraction + Nomic Vision | Medium | Medium |
| ColPali (page as image) | High | High |
| pymupdf4llm → markdown → TextChunker | Medium-high | Low |

**Recommended**: `pymupdf4llm` for text-heavy PDFs (preserves tables as markdown), ColPali for diagram-heavy PDFs.

## Image Passthrough Chunker

For raw images (not PDFs), the "chunk" is the image itself:

```python
class ImagePassthroughChunker(DocumentChunker):
    async def chunk(self, doc: Document) -> list[Chunk]:
        # Single chunk = entire image
        return [Chunk(
            id=f"{doc.id}:0",
            document_id=doc.id,
            content=doc.content,      # bytes
            chunk_index=0,
            start_char=0,
            end_char=len(doc.content),
            metadata={"is_image": True, "mime_type": doc.mime_type}
        )]
```

The `Embedder` detects `chunk.metadata["is_image"]` and routes to `embed.image()` instead of `embed.text()`.

## Relations

- Implements → [[reference/rag/multimodal-rag-ingestion-patterns]]
- Related → [[projects/graphrag/architecture/graphrag Plugin Categories Registry]]
- Related → [[projects/graphrag/research/graphrag SOTA RAG Landscape 2026]]
- Related → [[projects/graphrag/research/graphrag Codebase Ingestion via AST]]

---
*Evolution Log*
- 2026-03-29: Initial creation from Layer 2 Agent B findings — Nomic, ColPali, MIME dispatch pattern