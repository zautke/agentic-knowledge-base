---
title: ADR-001 — agent-toolkit Dependencies
type: reference
permalink: agent-kb/projects/agent-toolkit/architecture/decisions/adr-001-agent-toolkit-dependencies
created: 2026-04-18
modified: 2026-04-18
entity_type: reference
status: evergreen
tags:
- project/agent-toolkit
- domain/architecture
- domain/decisions
- op/adr
---

# ADR-001: Dependency Architecture

## Context

`tavily-agent-toolkit` v0.1.0 has 30 indexed files and 190 symbols. This ADR documents the dependency health findings from jcodemunch quality analysis (2026-04-18).

---

## Dependency Cycle Analysis

**Result: 0 dependency cycles detected.**

The package has a clean acyclic dependency graph. The module hierarchy is:

```
models.py (ModelConfig, ModelObject, TavilyUsage, LLMUsage, ToolUsageStats)
    ↓
utilities/utils.py (ainvoke_with_fallback, generate_subqueries, format_web_results, ...)
    ↓
tools/ (search_and_answer, search_dedup, crawl_and_summarize, extract_and_summarize, social_media_search, search_and_format)
    ↓
agents/hybrid_researcher.py (HybridResearcher)
    ↓
use-cases/ (example programs — isolated, not imported by library code)
```

No circular imports. No bidirectional dependencies between library layers.

---

## Coupling Metrics

From jcodemunch `get_coupling_metrics` (2026-04-18):

| Metric | Value | Assessment |
|--------|-------|-----------|
| Dependency cycles | 0 | Excellent |
| Equal PageRank distribution | Yes | Flat topology — no single choke point |
| Hotspot density | Low | Only 3 files flagged as complexity hotspots |
| Tectonic drifters | 14 files | Use-cases + tests are isolated (expected) |

**Flat PageRank** means no single module is a dependency bottleneck. The models, utilities, tools, and agents layers each carry roughly equal centrality. This is appropriate for a library with 6 peer tools.

---

## Hotspot Files

From jcodemunch `get_hotspots` (2026-04-18):

| File | Reason |
|------|--------|
| `utilities/utils.py` | High symbol count — all utility functions in one file |
| `agents/hybrid_researcher.py` | Two-mode implementation with sub-agent orchestration |
| `tools/search_and_answer.py` | Multi-step pipeline with token guard logic |

These are manageable complexity concentrations, not anti-patterns. They reflect genuine domain complexity rather than bad decomposition.

---

## External Dependencies (Hard)

```toml
# pyproject.toml hard dependencies
langchain         # init_chat_model, BaseChatModel
langchain-core    # @tool, tool invocation
langgraph         # create_agent, StateGraph
tavily-python     # TavilyClient, AsyncTavilyClient
anthropic         # Anthropic SDK (use-cases)
pydantic          # BaseModel, structured output
python-dotenv     # env var loading
asyncio           # stdlib — async parallel execution
```

LangChain is the only opinionated external dependency. All LLM calls go through `init_chat_model` — this is a deliberate decision to remain provider-agnostic.

---

## Layer Violations

**Result: None detected.**

Use-cases import from the library (`tavily_agent_toolkit`), but the library does not import from use-cases. The tectonic map shows 14 "drifter" files — these are the use-case scripts and test files that are isolated example programs, not part of the library's call graph. This is architecturally correct.

---

## Decision: Keep Single-File Utilities

`utilities/utils.py` is large (many symbols), but splitting it would:
1. Introduce import complexity for consumers
2. Break the clean layering without adding clarity
3. Require updating many import paths in tools/agents

**Decision**: Accept `utils.py` as a utilities barrel — acceptable for v0.1.0 given the flat coupling graph.

---

## Relations

- Related: [[agent-toolkit — Architecture Index]]
- Related: [[ADR-000 — agent-toolkit Init]]

---

## Evolution Log

| Date | Change | Trigger |
|------|--------|---------|
| 2026-04-18 | Created from jcodemunch get_dependency_cycles + get_coupling_metrics + get_hotspots data | Phase 3-T5 ingestion |