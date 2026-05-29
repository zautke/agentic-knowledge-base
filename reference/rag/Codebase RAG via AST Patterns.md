---
title: Codebase RAG via AST Patterns
type: reference
permalink: agent-kb/reference/rag/codebase-rag-via-ast-patterns
created: 2026-03-29
modified: 2026-03-29
entity_type: reference
status: evergreen
tags:
- project/agent-kb
- domain/rag
- domain/code-intelligence
- domain/ast
- status/evergreen
---

# Codebase RAG via AST Patterns

Global reference for RAG systems that ingest software codebases as their primary corpus. Covers AST-based chunking, deterministic knowledge graph construction, and code-specific retrieval patterns.

## Key Finding: AST Graphs > LLM-Extracted KGs for Code

**Paper:** *Reliable Graph-RAG for Codebases: AST-Derived Graphs vs LLM-Extracted Knowledge Graphs* (arXiv 2601.08773)

- AST-derived graphs are **deterministic** — no hallucination, complete coverage
- LLM-extracted graphs introduce false positives and miss implicit dependencies
- AST approach achieves lower indexing cost (no LLM calls during ingestion)
- Multi-hop reasoning still works: controller → service → repository chains via call graph traversal

**When LLM extraction is better:** Conceptual/semantic relationships not expressed in AST syntax (e.g., "this class implements the Observer pattern"). Use hybrid: AST for structural edges, LLM for semantic edges.

## tree-sitter: Python Bindings (v0.25.2)

### Installation
```bash
pip install tree-sitter tree-sitter-languages   # pre-compiled wheels, 170+ languages
```

### Core API
```python
from tree_sitter import Language, Parser
import tree_sitter_python

PY_LANGUAGE = Language(tree_sitter_python.language())
parser = Parser(PY_LANGUAGE)

# Parse (must be bytes)
tree = parser.parse(b"def foo():\n    pass")

# Node access
node = tree.root_node
node.type          # "module", "function_definition", "class_definition", etc.
node.start_byte    # byte offset (not char offset — critical for UTF-8)
node.end_byte
node.start_point   # (row, col) tuple — 0-indexed
node.end_point
node.text          # bytes (decode to str)
node.children      # list[Node]
node.child_by_field_name("name")  # named field access
```

### v0.22+ Breaking Changes
| Old (v0.20) | New (v0.22+) |
|-------------|--------------|
| `Query.set_byte_range(range)` | `QueryCursor.set_byte_range(start, end)` |
| `Query.captures(node)` | `QueryCursor.captures(node)` |
| `node.text` → str | `node.text` → bytes |

### Extracting Function/Class Definitions
```python
def extract_definitions(tree, source_bytes):
    target_types = {"function_definition", "class_definition", "decorated_definition"}
    results = []
    
    def visit(node):
        if node.type in target_types:
            results.append({
                "type": node.type,
                "name": node.child_by_field_name("name").text.decode(),
                "start_line": node.start_point[0],
                "end_line": node.end_point[0],
                "content": source_bytes[node.start_byte:node.end_byte].decode("utf-8"),
            })
        for child in node.children:
            visit(child)
    
    visit(tree.root_node)
    return results
```

## AST Knowledge Graph Schema

For code entity extraction, the deterministic graph schema is:

**Entity types:**
- `Function` — name, file, line range, docstring, signature
- `Class` — name, file, line range, bases (inheritance)
- `Module` — file path, imports
- `Variable` — name, scope (local/global), type hint

**Relation types:**
- `CALLS` — function A calls function B
- `IMPORTS` — module A imports module/symbol B
- `INHERITS_FROM` — class A inherits from class B
- `DEFINES` — module/class defines function/class
- `USES` — function uses variable/constant

**Extraction logic:**
- `CALLS`: Find `call` nodes in function bodies; resolve callee name
- `IMPORTS`: Parse `import_statement` / `import_from_statement` nodes
- `INHERITS_FROM`: Parse `argument_list` of `class_definition`
- `DEFINES`: Structural hierarchy from AST

## Supported Languages (tree-sitter-languages)

170+ languages via pre-compiled wheels. Key ones for software repos:

Python, JavaScript, TypeScript, Go, Rust, Java, C, C++, C#, Ruby, PHP, Swift, Kotlin, Scala, Haskell, Lua, R, Julia, YAML, TOML, JSON, SQL, Dockerfile, Bash, HCL (Terraform), Proto, GraphQL

## Code-Specific Chunking Rules

1. **Never split inside a function** — a function body is the atomic unit for code RAG
2. **Preserve imports in first chunk** — import statements provide critical context for all chunks
3. **Include docstrings with their function/class** — docstrings are semantic metadata, not noise
4. **Chunk size by logical unit, not token count** — prefer function-level even if 800 tokens
5. **Fallback for unsupported languages** — RecursiveCharacterTextSplitter with code separators

## Code Retrieval Patterns

### Structural Query Pattern
"Find all classes that implement X" → graph query for `INHERITS_FROM` edges

### Semantic Query Pattern  
"How is authentication handled?" → vector search on docstrings + function signatures

### Multi-Hop Pattern
"What calls the database from the controller?" → vector find controller → graph traverse CALLS → reach database layer

### Hybrid Code Retrieval
Vector weight 0.35-0.65, graph weight 0.35-0.65, depth 2-3. Use `graphrag_local` profile for specific questions, `graphrag_drift` for architecture exploration.

## Key Repos

- **code-graph-rag** (vitali87): Full AST → KG pipeline, monorepo support — github.com/vitali87/code-graph-rag
- **rag-cli**: Claude Code integration, Chroma DB — github.com/ItMeDiaTech/rag-cli
- **Continue.dev**: IDE-integrated code RAG — docs.continue.dev/guides/custom-code-rag

## Relations

- part_of [[Hybrid RAG Architecture – SOTA 2025-2026]]
- related_to [[Multimodal RAG Ingestion Patterns]]
- related_to [[System Architecture – assist-indexer]]
- implemented_by [[graphrag Codebase Ingestion via AST]]

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-03-29 | Initial creation | Layer 2-A+B research: code-graph-rag, tree-sitter, AST patterns | graphrag plugin implementation sprint |
| 2026-05-28 | Repaired two broken relation links: `[[projects/indexer: System Architecture Overview]]` → `[[System Architecture – assist-indexer]]` and `[[graphrag: Codebase Ingestion and AST Research]]` → `[[graphrag Codebase Ingestion via AST]]` (matched to real note titles) | Links used placeholder/colon naming that never resolved | KB curation hygiene sweep (operations/reference partition) |