---
title: Multimodal RAG Ingestion Patterns
type: reference
permalink: agent-kb/reference/rag/multimodal-rag-ingestion-patterns
created: 2026-03-29
modified: 2026-03-29
entity_type: reference
status: evergreen
tags:
- project/agent-kb
- domain/rag
- domain/multimodal
- domain/embeddings
- status/evergreen
---

# Multimodal RAG Ingestion Patterns

Global reference for handling multiple modalities (text, code, images, PDFs) in a unified RAG ingestion pipeline.

## The Core Problem

A single corpus contains mixed modalities: Python files (`text/x-python`), Markdown docs (`text/markdown`), diagrams (`image/png`), research papers (`application/pdf`). A naive loader treats everything as plain text and loses structural meaning.

## MIME-Type Dispatch Pattern

The clean solution is a **dispatcher chunker** that routes by `Document.mime_type`:

```python
class MimeTypeDispatchChunker(Chunker):
    dispatch_table = {
        "text/plain":      PlainTextChunker,
        "text/markdown":   PlainTextChunker,
        "text/x-python":   ASTChunker(language="python"),
        "text/javascript": ASTChunker(language="javascript"),
        "image/png":       ImagePassthroughChunker,
        "image/jpeg":      ImagePassthroughChunker,
        "application/pdf": PDFPageChunker,
    }
    
    async def chunk(self, doc: Document) -> list[Chunk]:
        chunker = self.dispatch_table.get(doc.mime_type, PlainTextChunker)
        return await chunker.chunk(doc)
```

**Image encoding**: Store as `data:image/png;base64,<b64>` in `Chunk.content`. The embedder detects the data URI scheme and routes to vision model.

**PDF pages**: Render each page as PNG via PyMuPDF (pymupdf), then store as base64 image chunks with `metadata["pdf_page"] = page_num`.

## Embedding Model Matrix (2025-2026)

| Modality | Best Model | Dims | Notes |
|----------|-----------|------|-------|
| Plain text | Nomic Embed Text v1.5 | 768 | Free API, Ollama local, Matryoshka |
| Code | Nomic Embed Text v1.5 | 768 | Adequate; Voyage Code-3 superior but separate dims |
| Images | Nomic Embed Vision v1.5 | 3584 | Aligned with Text v1.5 — cross-modal search works |
| PDFs (layout-sensitive) | ColPali / colqwen2-v1.0 | multi-vector | Late-interaction; heavy storage (~256KB/page) |
| Documents (general) | Nomic Embed Multimodal 7B | 3584 | Single-vector, slower than ColPali |

**Voyage Code-3**: 97.3% MRR for code, 2048 dims — best-in-class for code-only systems but mismatches text embedding dims.

## Vector Space Strategy: Unified vs Separate

### Unified Space (Recommended Default)
- Nomic Text v1.5 (text+code) + Nomic Vision v1.5 (images) — both produce 768-3584 dim vectors in the same semantic space
- Cross-modal queries work: text query → finds relevant image chunks
- Single vector index, simple retrieval
- Acceptable precision loss for mixed corpora

### Separate Namespaces (High Precision)
- Code → `text_code` namespace; images → `image` namespace
- Query both, fuse results at retrieval time
- 2-3x storage; higher per-modality precision
- Use when corpus is predominantly one modality

### ColPali (PDF-Heavy Corpora)
- Separate multi-vector index for PDF page images
- Late-interaction scoring: 128 patch tokens × 1024 dims per page
- Best precision for visually complex documents (tables, figures)
- Prohibitive storage at scale (TB for millions of pages)
- Use only when PDF layout fidelity is critical

## Chunking Strategies by Modality

| MIME Type | Chunker | Boundary | Overlap |
|-----------|---------|----------|---------|
| text/plain | FixedSize or Semantic | 512 tokens | 64 tokens |
| text/markdown | RecursiveCharacter | Headers → paragraphs | 64 tokens |
| text/x-python | AST (tree-sitter) | Function/class defs | 5 lines |
| text/javascript | AST (tree-sitter) | Function/class defs | 5 lines |
| image/* | Identity | One chunk per image | None |
| application/pdf | PDFPage → Image | One chunk per page | None |

## Nomic Embed API Pattern

```python
from nomic import embed

# Text embedding
response = embed.text(
    texts=["my text"],
    model="nomic-embed-text-v1.5",
    task_type="search_document",  # or "search_query"
    dimensionality=768             # Matryoshka: 256–768
)

# Image embedding (aligned with text)
response = embed.image(
    images=[pil_image_or_path],
    model="nomic-embed-vision-v1.5"
)

# Both live in same semantic space → cross-modal search works
```

**Local via Ollama:** `ollama pull nomic-embed-text:v1.5` — zero API cost for local development.

## Key Gotchas

1. **tree-sitter v0.22+ breaking change**: `Query.set_byte_range()` moved to `QueryCursor.set_byte_range()`. `node.text` returns `bytes`, not `str` — always `.decode('utf-8')`.
2. **Chroma sync→async bridge**: `chromadb` client is synchronous; wrap all calls with `asyncio.get_event_loop().run_in_executor(None, lambda: ...)`.
3. **Qdrant point IDs**: Must be UUID or integer, not arbitrary strings. Hash or map chunk_ids.
4. **PDF rendering**: `pymupdf` (pip install pymupdf) is the fastest PDF→image renderer; use `page.get_pixmap(matrix=Matrix(2,2))` for 2x zoom.
5. **ColPali storage**: ~256KB per page × N pages. Do the math before committing.

## Relations

- part_of [[Hybrid RAG Architecture – SOTA 2025-2026]]
- related_to [[Codebase RAG via AST Patterns]]
- implemented_by [[graphrag: Multimodal Plugin Patterns]]

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-03-29 | Initial creation | Layer 2-B research: multimodal embedding and chunking | graphrag plugin implementation sprint |