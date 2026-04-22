---
title: agent-toolkit — crawl_and_summarize Tool
type: reference
permalink: agent-kb/projects/agent-toolkit/knowledge/tools/agent-toolkit-crawl-and-summarize-tool
created: 2026-04-18
modified: 2026-04-18
entity_type: reference
status: evergreen
tags:
- project/agent-toolkit
- domain/agent-engineering
- domain/web-crawl
- status/evergreen
---

# crawl_and_summarize Tool

## What It Does
Crawls a website (depth/breadth/limit controls), cleans the raw content, then uses an LLM to summarize. Supports instruction-guided crawls, structured output, and domain/path filtering.

**Source**: `tools/crawl_and_summarize.py::crawl_and_summarize` — `local/agent-toolkit-49aa8c6c`

## Signature
```python
async def crawl_and_summarize(
    url: str,
    model_config: ModelConfig,
    instructions: Optional[str] = None,   # LLM-guided extraction instructions
    output_schema: Optional[Type[BaseModel]] = None,
    chunks_per_source: int = 3,
    max_depth: int = 1,                   # crawl depth
    max_breadth: int = 20,                # pages per depth level
    limit: int = 50,                      # total page limit
    select_paths, exclude_paths,          # path filters
    select_domains, exclude_domains,      # domain filters
    allow_external: bool = True,
    extract_depth: str = "basic",
    timeout: float = 150,
    api_key: Optional[str] = None,
    max_retries: int = 1,
) -> dict
```

## Pipeline
```
1. TavilyClient.crawl(url, depth, breadth, limit, filters...)
2. For each crawled page:
   → WITH instructions: use raw_content directly (Tavily applies chunk filtering)
   → WITHOUT instructions: clean_raw_content() removes navigation/boilerplate
3. ainvoke_with_fallback(summary_prompt + all_pages_content)
   → Grounding constraint: "only include information explicitly present"
   → Structured output if output_schema provided
```

## Key Behavioral Difference: `instructions` parameter
- **With instructions**: `instructions` + `chunks_per_source` passed to crawl API — Tavily's backend applies chunk-based filtering. `raw_content` contains pre-filtered relevant chunks.
- **Without instructions**: Full page content returned. `clean_raw_content()` applied client-side to strip navigation, ads, and markdown artifacts.

## Output
```python
{
    "results": [{"url": ..., "raw_content": ..., ...}],  # per-page crawl data
    "summary": str | BaseModel,
    "usage": {"response_time": ..., "tavily": {"crawl_count": ..., ...}, "llm": {...}}
}
```

## Grounding Prompt Pattern
```python
grounding = "Only include information explicitly present in the content. 
             Do not fabricate or infer missing information..."
if output_schema:
    grounding += " For fields without supporting information, set it to null."
```

## Relations
- part_of [[agent-toolkit — README]]
- uses [[agent-toolkit — Observability Data Model]]