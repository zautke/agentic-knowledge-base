---
title: agent-toolkit — Observability Data Model
type: reference
permalink: agent-kb/projects/agent-toolkit/knowledge/agent-toolkit-observability-data-model
created: 2026-04-18
modified: 2026-04-18
entity_type: reference
status: evergreen
tags:
- project/agent-toolkit
- domain/observability
- domain/agent-engineering
- status/evergreen
---

# agent-toolkit — Observability Data Model

## Design Principle
Every tool call returns a `usage` dict that separately tracks Tavily API costs, LLM token costs, and wall-clock timing. Components only appear in output if they were actually used (sparse serialization).

**Source**: `models.py` — `local/agent-toolkit-49aa8c6c`

## Class Hierarchy

```
ToolUsageStats
├── response_time: float               # end-to-end wall clock
├── internal_function_response_time: float  # RAG/internal call time
├── tavily: TavilyUsage
│   ├── total_credits: int
│   ├── search_count / search_response_time
│   ├── extract_count / extract_response_time
│   └── crawl_count / crawl_response_time
└── llm: LLMUsage
    ├── total_input_tokens: int
    ├── total_output_tokens: int
    ├── llm_call_count: int
    └── llm_response_time: float
```

## Serialization (`to_dict`)
All three classes have `to_dict()` that excludes zero/unused fields:

```python
# TavilyUsage.to_dict() — only includes API types actually called
{"total_credits": 5, "search_count": 3, "search_response_time": 1.23}

# LLMUsage.to_dict()
{"total_input_tokens": 1200, "total_output_tokens": 300,
 "total_tokens": 1500, "llm_call_count": 2, "llm_response_time": 2.1}

# ToolUsageStats.to_dict() — only includes non-zero components
{"response_time": 4.5, "tavily": {...}, "llm": {...}}
```

## Merge Pattern
All usage classes support `.merge(other)` for aggregating across multiple tool calls in a pipeline:
```python
usage = ToolUsageStats()
# After each sub-call:
usage.tavily.add_search(credits=2, response_time=0.8)
usage.llm.merge(sub_call_llm_usage)
```

## Usage Collection Pattern in Tools
Tools force `include_usage=True` on all Tavily API calls (via `async_retry`/`search_with_retry`) to ensure credit data is always captured. LLM usage extracted via `_extract_llm_usage(raw_message, elapsed)` which reads from `response_metadata.token_usage`.

## Complexity Note
`ToolUsageStats.to_dict` has CC=11 due to the conditional sparse serialization logic. This is intentional but could be simplified with a more declarative approach.

## Relations
- part_of [[agent-toolkit — README]]
- related_to [[agent-toolkit — HybridResearcher Agent]]