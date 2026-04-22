---
title: agent-toolkit — search_and_format Tool
type: reference
permalink: agent-kb/projects/agent-toolkit/knowledge/tools/agent-toolkit-search-and-format-tool
created: 2026-04-18
modified: 2026-04-18
entity_type: reference
status: evergreen
tags:
- project/agent-toolkit
- domain/agent-engineering
- domain/web-search
- status/evergreen
---

# search_and_format Tool

## What It Does
Lightweight search tool that returns formatted string results without LLM synthesis. Used in chatbot and agentic workflows where the agent (not this tool) decides what to do with the results.

**Source**: `tools/search_and_format.py::search_and_format` — `local/agent-toolkit-49aa8c6c`

## Key Difference from `search_and_answer`
| | `search_and_format` | `search_and_answer` |
|---|---|---|
| LLM synthesis | ❌ None | ✅ Yes |
| Output | Formatted string | Answer + usage |
| Usage tracking | None (no LLM) | Full Tavily + LLM |
| Use case | Agent calls multiple times | Single Q&A |

## Signature
```python
async def search_and_format(
    queries: list[str],          # 1 = direct; N = parallel dedup
    api_key: str,
    threshold: float = 0.3,
    search_depth: str = "advanced",
    topic, time_range, start_date, end_date, days,
    max_results: int = 10,
    include_domains, exclude_domains,
    ...
) -> str   # formatted results string
```

## Routing Logic
```python
if len(queries) == 1:
    # TavilyClient.search() directly
else:
    # search_dedup() — parallel + deduplicated
```

## Pipeline
```
1. search (direct or dedup)
2. filter by score >= threshold (default 0.3)
3. sort by score, keep top 20
4. format_web_results() → structured SOURCE blocks
5. return formatted string (no LLM)
```

## Primary Use Case
Chatbot tool call — agent requests search, decides how many times to call, accumulates results across turns, synthesizes at the application level.

## Relations
- part_of [[agent-toolkit — README]]
- used_by [[agent-toolkit — async_search_and_dedup Tool]]