---
title: agent-toolkit — Context Engineering Pipeline
type: reference
permalink: agent-kb/projects/agent-toolkit/knowledge/agent-toolkit-context-engineering-pipeline
created: 2026-04-18
modified: 2026-04-18
entity_type: reference
status: evergreen
tags:
- project/agent-toolkit
- domain/context-engineering
- domain/agent-engineering
- status/evergreen
---

# agent-toolkit — Context Engineering Pipeline

## Overview
The `utilities/utils.py` module contains the context engineering layer — all the functions that prepare raw web content for LLM consumption. This is the "context as first-class" design principle made concrete.

**Source**: `utilities/utils.py` — `local/agent-toolkit-49aa8c6c`

## The Pipeline (in order of a typical tool call)

### 1. Parallel Search + Retry (`async_retry`, `search_with_retry`, `extract_with_retry`, `crawl_with_retry`)
Wrap all Tavily API calls with exponential backoff (`2^attempt` seconds). Force `include_usage=True` on every call — observability is non-negotiable.

### 2. Subquery Generation (`generate_subqueries`)
```python
async def generate_subqueries(
    query: str,
    model_config: ModelConfig,
    max_number_of_subqueries: int = 4,
    context: str | None = None,           # internal RAG output feeds here
    output_schema: Optional[Type[BaseModel]] = None,
) -> list[str]:
    # Prompt: "Generate up to N Google-style queries covering different subtopics"
    # With context: "fill the gaps in the context for this query"
    # With output_schema: "fill out this schema"
```
Context-aware subquery generation — the LLM sees what you already know and generates queries that fill gaps, not duplicates.

### 3. Deduplication (in `search_dedup`)
Deduplicate by URL, merge unique content chunks. See [[agent-toolkit — async_search_and_dedup Tool]].

### 4. Content Formatting (`format_web_results`)
```python
def format_web_results(results: Sequence[SearchResult]) -> str:
    # Formats as: --- SOURCE N: {title} ---\nURL: {url}\n\nSUMMARY OF WEBPAGE:\n{content}
```
Structured format that helps LLMs identify and cite sources reliably.

### 5. Content Cleaning (`clean_raw_content`)
Strips navigation menus, footer boilerplate, social media buttons, markdown artifacts. Applied when raw full-page content is returned (not chunk-filtered content).

### 6. Token Counting (`count_tokens`)
```python
def count_tokens(text: str, model: str = "cl100k_base") -> int:
    # Uses tiktoken — model-specific encoding
```
Used to guard against context window overflow before every LLM call.

### 7. Long Content Summarization (`summarize_long_content`)
```python
async def summarize_long_content(
    model_config: ModelConfig, content: str, query: str, token_limit: int = 40000
) -> str:
    # Groups sources into chunks (~40k input per chunk)
    # Summarizes each chunk with capped output tokens (token_limit / num_chunks)
    # Returns concatenated summaries with source attribution
```
Triggered when content > 80% of `token_limit`. Chunks are grouped to minimize LLM calls while staying within context.

### 8. Synthesis (`synthesize_results`)
```python
async def synthesize_results(
    query: str, model_config: ModelConfig, research_results: str,
    output_schema: Optional[Type[BaseModel]] = None,
    research_synthesis_prompt: Optional[str] = None,
) -> str | BaseModel:
    # Unstructured: "Synthesize into report with inline citations and Sources section"
    # Structured: "Extract info from results to populate schema; null if not found"
```

## Full Pipeline Visualization
```
Raw web results
    ↓ format_web_results()         → structured source blocks
    ↓ clean_raw_content()          → remove boilerplate (optional)
    ↓ count_tokens()               → guard check
    ↓ summarize_long_content()     → compress if > 80% token limit (optional)
    ↓ synthesize_results()         → final report or Pydantic object
```

## Relations
- part_of [[agent-toolkit — README]]
- implements [[Hybrid Intelligence Pattern]]
- used_by [[agent-toolkit — HybridResearcher Agent]]
- used_by [[agent-toolkit — search_and_answer Tool]]