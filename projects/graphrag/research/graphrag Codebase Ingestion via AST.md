---
title: graphrag Codebase Ingestion via AST
type: reference
permalink: agent-kb/projects/graphrag/research/graphrag-codebase-ingestion-via-ast
created: 2026-03-29
modified: 2026-03-29
entity_type: reference
status: evergreen
tags:
- project/graphrag
- domain/codebase-rag
- domain/ast
- domain/research
---

# graphrag Codebase Ingestion via AST

How to implement the `chunker`, `entity_extractor`, and `relation_extractor` plugins for code ingestion using tree-sitter AST parsing — deterministic, hallucination-free, 170+ language support.

## Core Finding: AST > LLM for Code KGs

LLM-extracted code knowledge graphs hallucinate relationships and miss structural details. tree-sitter AST parsing:
- Deterministic (same code → same graph, always)
- No API costs
- Captures call graphs, import chains, inheritance trees directly from syntax
- 170+ languages (Python, JS, Go, Rust, Java, C/C++, etc.)

## tree-sitter v0.25.2 API (Python)

**CRITICAL: v0.22 breaking changes** — pre-v0.22 tutorials are wrong.

```python
# pip install tree-sitter==0.25.2 tree-sitter-python
import tree_sitter_python as tspython
from tree_sitter import Language, Parser

PY_LANGUAGE = Language(tspython.language())

def parse_python(source: str):
    parser = Parser(PY_LANGUAGE)
    tree = parser.parse(source.encode())
    return tree.root_node
```

**Pre-v0.22 (BROKEN — do not use)**:
```python
# OLD — fails on v0.22+
Language.build_library('build/languages.so', ['vendor/tree-sitter-python'])
PY_LANGUAGE = Language('build/languages.so', 'python')
parser = Parser()
parser.set_language(PY_LANGUAGE)
```

### Node Traversal

```python
def walk_tree(node, results: list):
    if node.type in ('function_definition', 'class_definition'):
        name_node = node.child_by_field_name('name')
        if name_node:
            results.append({
                'type': node.type,
                'name': name_node.text.decode(),
                'start_byte': node.start_byte,
                'end_byte': node.end_byte,
                'start_point': node.start_point,   # (row, col)
                'end_point': node.end_point,
            })
    for child in node.children:
        walk_tree(child, results)
```

### Call Graph Extraction

```python
def extract_calls(node, scope: str = None) -> list[dict]:
    """Find all function calls within a function body."""
    calls = []
    if node.type == 'call':
        func_node = node.child_by_field_name('function')
        if func_node:
            calls.append({'caller': scope, 'callee': func_node.text.decode()})
    for child in node.children:
        calls.extend(extract_calls(child, scope))
    return calls
```

## AST Chunker Pattern (for `DocumentChunker` ABC)

Route by `document.mime_type == "text/x-python"` (or other code MIME types):

```python
class ASTChunker(DocumentChunker):
    plugin_category = "chunker"

    async def chunk(self, doc: Document) -> list[Chunk]:
        if doc.mime_type == "text/x-python":
            return await self._chunk_python(doc)
        # fallback to text chunker
        return await self._chunk_text(doc)

    async def _chunk_python(self, doc: Document) -> list[Chunk]:
        root = parse_python(doc.content)
        chunks = []
        for node in self._collect_top_level(root):
            chunks.append(Chunk(
                id=f"{doc.id}:{node['start_byte']}",
                document_id=doc.id,
                content=doc.content[node['start_byte']:node['end_byte']],
                chunk_index=len(chunks),
                start_char=node['start_byte'],
                end_char=node['end_byte'],
                metadata={
                    'node_type': node['type'],     # 'function_definition'
                    'symbol_name': node['name'],   # 'my_function'
                    'language': 'python',
                }
            ))
        return chunks
```

## Code Graph Schema (for `Entity` and `Relation` models)

### Entity Types for Code
| Entity Type | Source | Example |
|-------------|--------|---------|
| `FUNCTION` | `function_definition` node | `embed(texts: list[str])` |
| `CLASS` | `class_definition` node | `OpenAIEmbedder` |
| `MODULE` | `import` node | `numpy` |
| `FILE` | Document source | `embedder.py` |
| `VARIABLE` | `assignment` node (top-level only) | `DEFAULT_MODEL` |

### Relation Types for Code
| Relation Type | Source |
|---------------|--------|
| `CALLS` | `call` node inside function body |
| `IMPORTS` | `import_statement` node |
| `INHERITS_FROM` | `class_definition.bases` |
| `DEFINED_IN` | function/class → file |
| `BELONGS_TO` | function → class (if method) |

## Multi-Language Support

```python
# Install language grammars separately
# pip install tree-sitter-python tree-sitter-javascript tree-sitter-go
# ...

LANGUAGE_MAP = {
    "text/x-python": ("tree_sitter_python", "python"),
    "application/javascript": ("tree_sitter_javascript", "javascript"),
    "text/x-go": ("tree_sitter_go", "go"),
    "text/x-rust": ("tree_sitter_rust", "rust"),
    "text/x-java": ("tree_sitter_java", "java"),
}
```

## Integration with graphrag Pipeline

Stage 1 (ingestion) plugin chain for a codebase source:
1. `codebase_loader` → walks git repo, emits `Document` per file with `mime_type` set from extension
2. `text_preprocessor` → normalizes encoding, strips binary files
3. `ast_chunker` → routes by `mime_type`, produces function/class-level Chunks with AST metadata
4. `nomic_embedder` (or `openai_embedder`) → embeds code + docstrings in unified space

Stage 2 (graph_build) plugin chain for code:
1. `ast_entity_extractor` → reads `chunk.metadata['node_type']` + `chunk.metadata['symbol_name']`, produces typed Entities
2. `ast_relation_extractor` → call graph + imports, produces typed Relations
3. `networkx_graph_store` (dev) / `neo4j_graph_store` (prod) → persists KG

## Relations

- Implements → [[reference/rag/codebase-rag-via-ast-patterns]]
- Related → [[projects/graphrag/architecture/graphrag Plugin Categories Registry]]
- Related → [[projects/graphrag/research/graphrag SOTA RAG Landscape 2026]]
- Related → [[reference/rag/multimodal-rag-ingestion-patterns]]

---
*Evolution Log*
- 2026-03-29: Initial creation from Layer 2 Agent B findings and tree-sitter API research