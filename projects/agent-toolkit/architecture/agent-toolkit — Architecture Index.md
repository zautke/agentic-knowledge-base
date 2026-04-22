---
title: agent-toolkit — Architecture Index
type: reference
permalink: agent-kb/projects/agent-toolkit/architecture/agent-toolkit-architecture-index
created: 2026-04-18
modified: 2026-04-18
entity_type: reference
status: evergreen
tags:
- project/agent-toolkit
- domain/agent-engineering
- domain/architecture
- status/evergreen
---

# Tavily Agent Toolkit — Architecture

## Package Topology

```
agent-toolkit/
├── models.py              # Type system: ModelConfig, ModelObject, TavilyUsage, LLMUsage, ToolUsageStats
├── agents/
│   └── hybrid_researcher.py  # HybridResearcher: fast_mode + multi_agent_mode
├── tools/
│   ├── search_and_answer.py
│   ├── search_and_format.py
│   ├── async_search_and_dedup.py
│   ├── crawl_and_summarize.py
│   ├── extract_and_summarize.py
│   └── social_media.py
├── utilities/
│   ├── utils.py           # Retry, fallback, token counting, content cleaning, synthesis
│   └── research_stream.py # SSE streaming for Research API
└── use-cases/
    ├── claude_sdk/        # Anthropic SDK implementations
    └── langgraph/         # LangGraph implementations
```

## Tectonic Map (jcodemunch Phase 0)
- **2 logical plates** detected via behavioral coupling signals
- **14 drifter files** — files whose physical directory doesn't match their logical module:
  - `__init__.py`, `agents/__init__.py`, `agents/hybrid_researcher.py`, `models.py`, `tools/__init__.py`
  - All 6 use-case files (`claude_sdk/` + `langgraph/`)
  - `utilities/__init__.py`, `utilities/research_stream.py`, `utilities/utils.py`
- **Implication**: The code is logically cohesive but physically organized around API convenience, not module ownership. Use-cases are deliberately isolated (they're example programs, not library code).

## Dependency Health
- **Dependency cycles**: 0 — clean layered architecture
- **Import direction**: `use-cases` → `tools/agents` → `utilities` → `models` (correct one-way flow)

## Complexity Hotspots
| Symbol | File | Cyclomatic Complexity |
|---|---|---|
| `multi_agent_mode` | `agents/hybrid_researcher.py` | 12 |
| `ToolUsageStats.to_dict` | `models.py` | 11 |
| `OutputSchema.to_tavily_schema` | `models.py` | 7 |
| `fast_mode` | `agents/hybrid_researcher.py` | 5 |
| `hybrid_research` | `agents/hybrid_researcher.py` | 5 |

## Test Coverage
- **0% heuristic reachability** (66/66 non-test symbols unreached)
- Tests are integration tests requiring real `TAVILY_API_KEY` + `OPENAI_API_KEY` — this is intentional (see Testing Runbook)
- All 6 tools have dedicated test files; hybrid_researcher + models do not

## ADRs
- [[ADR-000 — agent-toolkit Init]] — Initial project setup
- [[ADR-001 — Dependencies and Coupling]] — Planned

## Relations
- part_of [[agent-toolkit — README]]
- related_to [[agent-toolkit — Gated KB Submission Protocol]]