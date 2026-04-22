---
title: agent-toolkit — async_search_and_dedup Tool
type: reference
permalink: agent-kb/projects/agent-toolkit/knowledge/tools/agent-toolkit-async-search-and-dedup-tool
created: 2026-04-18
modified: 2026-04-18
entity_type: reference
status: evergreen
tags:
- project/agent-toolkit
- domain/agent-engineering
- domain/web-search
- domain/context-engineering
- status/evergreen
---

# async_search_and_dedup Tool

## What It Does
Fires multiple Tavily searches in parallel using `AsyncTavilyClient` + `asyncio.gather`, then deduplicates results by URL — merging unique content chunks from the same source.

**Source**: `tools/async_search_and_dedup.py` — `local/agent-toolkit-49aa8c6c`

## Key Behavior
- **Parallel execution**: All queries run simultaneously via `asyncio.gather`
- **Deduplication**: Same URL appearing in multiple query results → merged, not duplicated
- **Chunk merging**: Content chunks (`" [...] "` separator) are split, deduped as a set, and rejoined
- **Score preservation**: Keeps the highest score when a URL appears multiple times
- **Usage aggregation**: `TavilyUsage.add_search()` called for each parallel response, then merged

## Deduplication Algorithm (`_deduplicate_by_url`)
```python
url_data: dict[str, tuple[dict, set[str]]] = {}

for response in search_responses:
    for result in response["results"]:
        url = result["url"]
        content = result["content"]
        chunks = {c for c in content.split(" [...] ") if c.strip()}
        
        if url in url_data:
            existing_result, existing_chunks = url_data[url]
            existing_chunks.update(chunks)          # merge unique chunks
            if score > existing_score:              # keep highest score
                existing_result["score"] = score
        else:
            url_data[url] = (result.copy(), chunks)

# Final: sort by score descending
results.sort(key=lambda x: x["score"], reverse=True)
```

## Usage Pattern
```python
results = await search_dedup(
    api_key="...",
    queries=["query 1", "query 2", "query 3"],
    max_results=5,
    search_depth="advanced",
    chunks_per_source=3,
)
# results["results"] = deduplicated, sorted by score
# results["tavily_usage"] = aggregated usage across all queries
```

## Output Schema
```python
{
    "results": [...],         # deduplicated, sorted by score
    "images": [...] | None,
    "answer": str | None,
    "response_time": float,   # wall clock (max of parallel requests)
    "queries": [str, ...],    # original query list
    "tavily_usage": {...}     # aggregated TavilyUsage dict
}
```

## Note on `chunks_per_source`
Only active when `search_depth="advanced"`. Advanced search returns content as multiple chunks per URL; the dedup logic preserves unique chunk content across queries pointing to the same URL.

## Relations
- part_of [[agent-toolkit — README]]
- used_by [[agent-toolkit — search_and_answer Tool]]
- used_by [[agent-toolkit — HybridResearcher Agent]]