---
title: agent-toolkit — extract_and_summarize Tool
type: reference
permalink: agent-kb/projects/agent-toolkit/knowledge/tools/agent-toolkit-extract-and-summarize-tool
created: 2026-04-18
modified: 2026-04-18
entity_type: reference
status: evergreen
tags:
- project/agent-toolkit
- domain/agent-engineering
- domain/web-extract
- status/evergreen
---

# extract_and_summarize Tool

## What It Does
Extracts content from specific URLs (max 20 per call) and summarizes each page individually using an LLM. Supports query-focused chunk extraction and structured Pydantic output.

**Source**: `tools/extract_and_summarize.py::extract_and_summarize` — `local/agent-toolkit-49aa8c6c`

## Signature
```python
async def extract_and_summarize(
    urls: list[str],                        # max 20 URLs per call
    model_config: ModelConfig,
    query: Optional[str] = None,            # if set: chunk-based relevant extraction
    output_schema: Optional[Type[BaseModel]] = None,
    chunks_per_source: int = 5,             # only when query provided
    extract_depth: Literal["basic","advanced"] = "basic",
    include_images: bool = False,
    format: Literal["markdown","text"] = "markdown",
    timeout: Optional[float] = None,
    api_key: Optional[str] = None,
    max_retries: int = 1,
) -> dict
```

## Two-Mode Pipeline

### Without `query` (full page extraction):
```
1. TavilyClient.extract(urls, extract_depth="basic"...)
2. clean_raw_content(item["raw_content"])   # remove boilerplate
3. LLM: "Summarize this content" + grounding constraint
```

### With `query` (chunk-based focused extraction):
```
1. TavilyClient.extract(urls, query=query, chunks_per_source=5...)
   → Tavily reranks chunks by query relevance
   → raw_content contains only relevant chunks
2. LLM: "Summarize based on this query: {query}" + chunks
```

## Key Design: Per-URL Summarization
Unlike `crawl_and_summarize` (single LLM call for all pages), this tool calls the LLM once **per URL**. Suitable for targeted extraction where each URL needs independent analysis.

## Output
```python
{
    "results": [
        {
            "url": ...,
            "title": ...,
            "summary": str | BaseModel,  # added; raw_content removed
            # other tavily fields
        },
        ...
    ],
    "usage": {"response_time": ..., "tavily": {"extract_count": ...}, "llm": {...}}
}
```

## Recommended Two-Step Pattern (from CLAUDE.md)
```python
# Step 1: Search and score
search_results = await search_and_answer(query=..., ...)
# Step 2: Extract high-confidence URLs only
high_confidence = [r["url"] for r in search_results["results"] if r["score"] > 0.5]
extracted = await extract_and_summarize(urls=high_confidence, query=query, ...)
```

## Relations
- part_of [[agent-toolkit — README]]
- uses [[agent-toolkit — Observability Data Model]]