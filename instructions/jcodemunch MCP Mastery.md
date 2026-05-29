---
title: jcodemunch MCP Mastery
type: instruction
permalink: agent-kb/instructions/jcodemunch-mcp-mastery
status: active
tags:
- jcodemunch
- mcp
- code-navigation
- token-efficiency
- ast
- tools
- source/largo
- machine/largo
---

# jcodemunch MCP Mastery

## Core Philosophy

jcodemunch-mcp is a **precision code retrieval engine** built on tree-sitter AST (Abstract Syntax Tree) parsing. It solves the fundamental LLM inefficiency of reading entire files to find one symbol.

**Core principle:** "Retrieval precision > brute-force context expansion"

**What it is:** File-level syntactic analysis via tree-sitter AST — 70+ languages, no compilation required.  
**What it is NOT:** Semantic analysis (cannot resolve interface implementations, type inference, or complex inheritance chains — that requires LSP).

**Token savings benchmark:**
- Single function lookup: 80.7% savings (7,500 → 1,449 tokens)
- FastAPI full repo: 99.8% savings (214,312 → 480 tokens)
- Production A/B test (Vue 3 + Firebase): 80% success rate vs 72% with native tools

---

## Setup & Indexing (Critical First Steps)

```
resolve_repo({ "path": "." })          # Verify repo is indexed
index_folder({ "path": ".", "incremental": true })  # If not indexed
register_edit({ "file_paths": [...] })  # After any manual file edit
index_file({ "path": "/abs/path/to/file" })  # After editing a file
```

**Configuration files:**
- Global: `~/.code-index/config.jsonc`
- Project: `<project>/.jcodemunch.jsonc`
- Key limits: `max_folder_files` (default 500/folder), `max_index_files` (10,000 hard cap)

---

## 5 Core Workflows

### 1. Explore Unfamiliar Repository
```
suggest_queries()                              # Get search ideas
get_repo_outline()                             # Understand structure
get_file_tree({ "path_prefix": "src" })       # Navigate
get_file_outline({ "file_path": "..." })      # Before any file read
search_symbols({ "query": "...", "kind": "function" })
```

### 2. Targeted Edit (Single Function)
```
search_symbols({ "query": "myFunction" })     # Find it
get_file_outline({ "file_path": "..." })      # See context
get_symbol_source({ "symbol_id": "..." })     # Read exact code
get_blast_radius({ "symbol": "...", "depth": 2 })  # Impact
plan_refactoring({ "symbol": "...", "refactor_type": "rename", "new_name": "..." })
```

### 3. Code Quality Audit
```
get_hotspots({ "top_n": 20 })                 # Risk ranking (complexity × churn)
get_dead_code_v2()                             # Unreachable code (triple-signal)
get_untested_symbols()                         # Coverage gaps
get_dependency_cycles()                        # Circular imports
```

### 4. Change Impact Analysis (PRs, Refactoring)
```
get_changed_symbols({ "since_sha": "..." })   # Map diff to symbols
get_blast_radius({ "symbol": "...", "include_source": true })
get_pr_risk_profile({ "base_ref": "main", "head_ref": "..." })
get_call_hierarchy({ "symbol_id": "...", "depth": 3 })
```

### 5. Architecture Review
```
get_symbol_importance({ "algorithm": "pagerank", "top_n": 20 })
get_tectonic_map()                             # Module topology
get_layer_violations()                         # Architecture boundary crossing
get_coupling_metrics({ "module_path": "..." })  # Ca/Ce/Instability
get_dependency_cycles()
```

---

## Critical Rules

1. **ALWAYS** `get_file_outline` before `get_file_content`
2. **ALWAYS** `search_symbols` before `get_symbol_source`
3. Use `get_context_bundle` for 2+ related symbols (not repeated single-symbol calls)
4. Call `register_edit` after every manual file edit
5. **Never read entire files** — retrieve at symbol level
6. Use `plan_turn` at task start for confidence assessment + recommended symbols

---

## Key Tool Parameters (Top 10 Most-Used)

### `search_symbols`
| Param | Values | Notes |
|-------|--------|-------|
| `query` | string | Name, signature, summary, docstring |
| `kind` | function/class/method/constant/type | Filter |
| `language` | python/typescript/java/etc | Filter |
| `file_pattern` | glob | e.g. "src/**/*.py" |
| `detail_level` | compact/standard/full | compact=15 tokens each |
| `sort_by` | relevance/centrality/combined | combined=PageRank + BM25 |
| `fuzzy` | bool | Levenshtein tolerance |
| `semantic` | bool | Requires embedding provider |
| `token_budget` | int | Hard cap, greedy packing |

### `get_context_bundle`
| Param | Values | Notes |
|-------|--------|-------|
| `symbol_ids` | array | Batch multiple symbols |
| `token_budget` | int | Truncate to fit |
| `budget_strategy` | most_relevant/core_first/compact | |
| `include_callers` | bool | Files that import this module |
| `output_format` | json/markdown | markdown=paste-ready doc |

### `get_blast_radius`
| Param | Values | Notes |
|-------|--------|-------|
| `symbol` | string | Symbol name or ID |
| `depth` | 1-3 | Import hops |
| `include_source` | bool | Add source snippets |
| `call_depth` | 0-3 | Also find calling symbols |
| `include_depth_scores` | bool | Per-depth risk scores |

### `search_ast` (Pattern Matching — 70+ Languages)
**Presets:** `empty_catch`, `bare_except`, `deeply_nested`, `nested_loops`, `god_function`, `eval_exec`, `hardcoded_secret`, `todo_fixme`, `magic_number`, `reassigned_param`

**Custom patterns:** `call:*.unwrap`, `string:/password/i`, `comment:/TODO/i`, `nesting:5+`, `loops:3+`, `lines:80+`

### `get_ranked_context` (Smart Context Assembly)
| Param | Values | Notes |
|-------|--------|-------|
| `query` | string | Natural language task description |
| `token_budget` | int | Default 4000 |
| `strategy` | combined/bm25/centrality | |
| `scope` | glob | Limit search area |

---

## Gotchas & Limitations

| Issue | Impact | Mitigation |
|-------|--------|-----------|
| Tree-sitter is syntactic, not semantic | Can't resolve interface implementations or type inference | Use `search_text` for framework patterns (Django decorators, Spring annotations) |
| Monorepo scaling cap | ~500 files/folder — large repos silently exclude subtrees | Use `path_prefix` to index subdirectories separately |
| Index staleness | Manual edits without re-indexing cause symbol mismatches | `register_edit` or `index_file` after every edit |
| Cross-repo semantic queries | `cross_repo=true` only works at import-graph level | Cannot resolve symbols across packages (LSP would) |
| Never log to stderr | Breaks MCP protocol | Use `--log-file` or `JCODEMUNCH_LOG_FILE` |

---

## Session Tracking Tools

```
get_session_context()      # Files accessed, searches, edits this session
get_session_snapshot()     # ~200 token compact summary for context continuity
get_session_stats()        # Token savings tracking (per-call + cumulative)
```

All responses include `_meta.tokens_saved` — cumulative savings persist to `~/.code-index/_savings.json`.

---

## Related Notes
- [[jcodemunch Tool Reference]] — full 60+ tool inventory with parameters
- [[Claude Skill Authoring Mastery]] — for writing the using-jcodemunch skill

---

## Evolution Log
- 2026-04-17: Created from deep research (GitHub README, USER_GUIDE.md, SPEC.md, ARCHITECTURE.md). Source: github.com/jgravelle/jcodemunch-mcp