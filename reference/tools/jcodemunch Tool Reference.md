---
title: jcodemunch Tool Reference
type: reference
permalink: agent-kb/reference/tools/jcodemunch-tool-reference
status: active
tags:
- jcodemunch
- mcp
- tools
- reference
- code-navigation
---

# jcodemunch Tool Reference

Full inventory of all 60+ jcodemunch MCP tools, grouped by workflow category. All tools prefixed `mcp__jcodemunch__` in Claude Code.

See [[jcodemunch MCP Mastery]] for workflows and usage patterns.

---

## Category 1: Indexing & Repository Management

| Tool | Purpose | Key Params |
|------|---------|-----------|
| `index_repo` | Index GitHub repo by URL | `url` |
| `index_folder` | Index local directory | `path`, `incremental` (bool) |
| `index_file` | Reindex single file after edit | `path` |
| `invalidate_cache` | Force full re-index | `repo` |
| `list_repos` | List all indexed repositories | — |
| `resolve_repo` | Resolve filesystem path → repo ID | `path` |
| `summarize_repo` | AI summarization of all symbols (batch) | `repo` |

---

## Category 2: Discovery & Orientation

| Tool | Purpose | Key Params |
|------|---------|-----------|
| `suggest_queries` | Get suggested search terms for repo | `repo` |
| `get_repo_outline` | High-level repo structure overview | `repo` |
| `get_file_tree` | File hierarchy | `repo`, `path_prefix` (glob filter) |
| `get_file_outline` | All symbols + signatures in file | `repo`, `file_path` or `file_paths[]` (batch) |
| `get_project_intel` | Parse non-code: Dockerfiles, CI, K8s, Makefiles, package.json | `repo` |

**Recommended first-time sequence:** `suggest_queries` → `get_repo_outline` → `get_file_tree`

---

## Category 3: Symbol Search & Retrieval (Core)

| Tool | Purpose | Key Params |
|------|---------|-----------|
| `search_symbols` | BM25 + semantic/fuzzy search | `repo`, `query`, `kind`, `language`, `file_pattern`, `decorator`, `max_results`, `token_budget`, `detail_level` (compact/standard/full), `fuzzy`, `semantic`, `sort_by` (relevance/centrality/combined) |
| `search_text` | Full-text / regex search (grep-equivalent) | `repo`, `query`, `is_regex`, `file_pattern`, `context_lines`, `max_results` |
| `get_symbol_source` | Exact symbol implementation | `repo`, `symbol_id`, `context_lines`, `verify` (hash check) |
| `get_context_bundle` | Symbol + related imports + deps (deduped) | `repo`, `symbol_id` or `symbol_ids[]`, `token_budget`, `budget_strategy` (most_relevant/core_first/compact), `include_callers`, `output_format` (json/markdown) |
| `get_file_content` | Raw file content with line range | `repo`, `file_path`, `start_line`, `end_line` |

**Rule:** Use `search_symbols` → `get_file_outline` → `get_symbol_source` (never jump straight to `get_file_content`).

---

## Category 4: Structural Analysis (Grep Cannot Answer These)

| Tool | Purpose | Key Params |
|------|---------|-----------|
| `find_references` | All files importing/referencing a symbol | `repo`, `identifier` or `identifiers[]`, `include_call_chain` |
| `find_importers` | Files that import a specific file | `repo`, `file_path`, `cross_repo` |
| `check_references` | Quick boolean: is identifier referenced? | `repo`, `identifier` or `identifiers[]` (batch), `search_content` |
| `find_dead_code` | Symbols unreachable from entry points | `repo`, `granularity` (symbol/file), `min_confidence` (0.8-1.0), `entry_point_patterns` |
| `get_dead_code_v2` | Triple-signal dead code (more reliable) | `repo`, `granularity`, `min_confidence` |

---

## Category 5: Impact & Dependency Analysis

| Tool | Purpose | Key Params |
|------|---------|-----------|
| `get_blast_radius` | What breaks if you change this? | `repo`, `symbol`, `depth` (1-3), `include_source`, `cross_repo`, `call_depth`, `include_depth_scores` |
| `get_call_hierarchy` | Callers and callees N levels deep | `repo`, `symbol_id`, `direction` (callers/callees/both), `depth` (1-5) |
| `get_impact_preview` | Complete caller chain for removal/rename | `repo`, `symbol` |
| `get_dependency_graph` | File-level import relationships | `repo`, `file`, `direction` (imports/importers/both), `depth` (1-3), `cross_repo` |
| `get_dependency_cycles` | Circular import detection (SCCs) | `repo` |

---

## Category 6: Code Quality Signals

| Tool | Purpose | Key Params |
|------|---------|-----------|
| `get_churn_rate` | Git history: commits, authors, volatility | `repo`, `target` (file or symbol), `days` (default 90) |
| `get_hotspots` | High-risk symbols (complexity × log(1 + commits)) | `repo`, `top_n`, `min_complexity`, `days` |
| `get_symbol_complexity` | Cyclomatic complexity, nesting, param count | `repo`, `symbol_id` |
| `get_untested_symbols` | Functions with no test file references | `repo`, `file_pattern`, `min_confidence`, `max_results` |
| `get_coupling_metrics` | Ca/Ce coupling + instability (I = Ce/(Ca+Ce)) | `repo`, `module_path` |

**Instability score:** 0 = stable (many dependents, few dependencies), 1 = unstable (many deps, few dependents)

---

## Category 7: Architecture & Design

| Tool | Purpose | Key Params |
|------|---------|-----------|
| `get_symbol_importance` | Most architecturally central (PageRank) | `repo`, `algorithm` (pagerank/degree), `scope`, `top_n` |
| `get_class_hierarchy` | Full inheritance chains (extends/implements) | `repo`, `class_name` |
| `get_tectonic_map` | Logical module topology (structural + behavioral + temporal) | `repo` |
| `get_layer_violations` | Architecture boundary crossing detection | `repo`, `rules` (or reads `.jcodemunch.jsonc`) |
| `get_cross_repo_map` | Cross-repository dependency mapping | `repos[]` |

**Tectonic map signals:** structural (imports) + behavioral (shared references) + temporal (git co-churn)

---

## Category 8: Advanced Analysis

| Tool | Purpose | Key Params |
|------|---------|-----------|
| `get_changed_symbols` | Map git diff to affected symbols | `repo`, `since_sha`, `until_sha`, `include_blast_radius` |
| `get_symbol_provenance` | Complete authorship lineage through git history | `repo`, `symbol`, `max_commits` |
| `get_pr_risk_profile` | Unified PR risk (blast radius + complexity + churn + test gaps) | `repo`, `base_ref`, `head_ref`, `days` |
| `get_signal_chains` | How external signals (HTTP/CLI/events) propagate through code | `repo`, `symbol` (optional), `kind`, `max_depth` |
| `search_ast` | Cross-language pattern matching (70+ languages) | `repo`, `pattern` (preset or custom), `category`, `file_pattern`, `language`, `max_results` |
| `winnow_symbols` | Multi-axis constraint query (AND intersection) | `repo`, `criteria[]`, `max_results`, `rank_by` |

**search_ast preset patterns:** `empty_catch`, `bare_except`, `deeply_nested`, `nested_loops`, `god_function`, `eval_exec`, `hardcoded_secret`, `todo_fixme`, `magic_number`, `reassigned_param`

**search_ast categories:** `security`, `error_handling`, `complexity`, `performance`, `maintenance`, `all`

---

## Category 9: Context Assembly & Optimization

| Tool | Purpose | Key Params |
|------|---------|-----------|
| `get_ranked_context` | Best-fit context within token budget | `repo`, `query`, `token_budget` (default 4000), `scope`, `strategy` (combined/bm25/centrality), `fusion`, `include_kinds` |
| `get_context_bundle` | Multi-symbol bundle with deduped imports | (see Category 3) |
| `get_session_context` | Files accessed, searches, edits this session | `max_files`, `max_queries` |
| `get_session_snapshot` | ~200 token compact session summary | `include_negative_evidence`, `max_edits`, `max_files`, `max_searches` |

---

## Category 10: Utility & Metadata

| Tool | Purpose | Key Params |
|------|---------|-----------|
| `plan_turn` | Analyze query → recommend symbols/files, gauge confidence | `repo`, `query`, `max_recommended` |
| `plan_refactoring` | Generate edit-ready refactoring instructions | `repo`, `symbol`, `refactor_type` (rename/move/extract/signature), `new_name`, `new_file`, `new_signature` |
| `check_rename_safe` | Detect name collisions before rename | `repo`, `symbol_id`, `new_name` |
| `get_related_symbols` | Symbols related via co-location/shared importers/name overlap | `repo`, `symbol_id`, `max_results` |
| `register_edit` | Invalidate caches after manual edits | `repo`, `file_paths[]`, `reindex` |
| `get_session_stats` | Token savings tracking (cumulative) | — |
| `render_diagram` | Convert graph output to Mermaid markup | raw output from call_hierarchy/blast_radius, `theme` (flow/risk/minimal) |
| `search_columns` | Column metadata search (dbt, database catalogs) | `repo`, `query`, `model_pattern`, `max_results` |
| `embed_repo` | Precompute symbol embeddings for semantic search | `repo`, `batch_size`, `force` |
| `audit_agent_config` | Audit config files for token waste / stale refs | `project_path`, `repo` |
| `get_extraction_candidates` | Functions good for extraction (complexity × callers) | `repo`, `file_path`, `min_complexity`, `min_callers` |

---

## Evolution Log
- 2026-04-17: Created from github.com/jgravelle/jcodemunch-mcp (README, USER_GUIDE.md, SPEC.md, CONFIGURATION.md, AGENT_HOOKS.md)