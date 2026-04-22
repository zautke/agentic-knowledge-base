---
title: agent-toolkit — search_and_answer Tool
type: reference
permalink: agent-kb/projects/agent-toolkit/knowledge/tools/agent-toolkit-search-and-answer-tool
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

# search_and_answer Tool

## What It Does
Executes web search, applies score/token filtering, and uses an LLM to synthesize a final answer. Supports subquery generation, structured Pydantic output, citation formatting, and image inclusion.

**Source**: `tools/search_and_answer.py::search_and_answer` — `local/agent-toolkit-49aa8c6c`

## Signature
```python
async def search_and_answer(
    query: str,
    api_key: str,
    model_config: ModelConfig,
    token_limit: int = 50000,
    output_schema: Optional[Type[BaseModel]] = None,
    max_number_of_subqueries: Optional[int] = None,  # None = single search
    threshold: Optional[float] = 0.3,                # score filter
    search_depth: Literal["basic","advanced"] = "advanced",
    topic: Optional[Literal["general","news","finance"]] = None,
    time_range, start_date, end_date, days,           # temporal filters
    max_results: int = 10,
    include_domains, exclude_domains,                  # domain filtering
    include_raw_content, include_images,               # content options
    include_citations: bool = False,
    max_retries: int = 1,
    **kwargs
) -> Dict[str, Any]
```

## Pipeline
```
1. (optional) generate_subqueries(query, context=output_schema)
   → if 1 subquery: single search_with_retry
   → if N subqueries: search_dedup (parallel, deduplicated)
2. filter by score >= threshold (default 0.3)
3. sort by score, keep top 20
4. format_web_results → clean_formatted_output
5. (if tokens > 80% of token_limit) summarize_long_content
6. ainvoke_with_fallback(answer_prompt + results)
   → with citations: numbered [1][2] inline + Sources section
   → with output_schema: structured Pydantic output
```

## Output
```python
{
    **tavily_search_result,   # original search response fields
    "answer": str | BaseModel,
    "usage": {"response_time": ..., "tavily": {...}, "llm": {...}}
}
```

## Key Design Details
- **Score threshold**: Default 0.3 — results below this score are dropped before synthesis
- **Token guard**: If formatted output > 80% of `token_limit`, triggers `summarize_long_content`
- **Subquery routing**: 1 subquery → direct search; N subqueries → `search_dedup` parallel path
- **Citation prompt**: Injects numbered citation instructions into the answer prompt when `include_citations=True`

## Relations
- part_of [[agent-toolkit — README]]
- uses [[agent-toolkit — Observability Data Model]]
- related_to [[agent-toolkit — async_search_and_dedup Tool]]