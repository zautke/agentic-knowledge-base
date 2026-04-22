---
title: agent-toolkit — HybridResearcher Agent
type: reference
permalink: agent-kb/projects/agent-toolkit/knowledge/agents/agent-toolkit-hybrid-researcher-agent
created: 2026-04-18
modified: 2026-04-18
entity_type: reference
status: evergreen
tags:
- project/agent-toolkit
- domain/agent-engineering
- domain/rag
- status/evergreen
---

# agent-toolkit — HybridResearcher Agent

## What It Does
Combines internal RAG (your vector store) with real-time web search into a single research call. Returns a synthesized report + web sources + full usage metrics.

**Source**: `agents/hybrid_researcher.py` — `local/agent-toolkit-49aa8c6c`

## Entry Point
```python
async def hybrid_research(
    api_key: str,
    query: str,
    model_config: ModelConfig,
    internal_rag_function: Callable[[str], str],  # your RAG function
    mode: Literal["fast", "multi_agent"] = "fast",
    output_schema: Optional[Type[OutputSchema]] = None,
    research_synthesis_prompt: Optional[str] = None,
) -> dict[str, Any]:
    # Routes to fast_mode or multi_agent_mode
```

## Mode 1: fast_mode (CC=5)

**Flow**:
```
1. internal_rag_function(query)         → internal context string
2. generate_subqueries(query, context)  → list of 5 web search queries
3. search_dedup(api_key, subqueries)    → deduplicated web results
4. format_web_results + clean           → LLM-ready context
5. synthesize_results(internal + web)  → final report
```

**Key design**: Subqueries are generated with internal context as input — the LLM knows what you already have, and generates web searches that fill the gaps.

**Output**:
```python
{"report": str | dict, "web_sources": [{"title": ..., "url": ...}], "usage": {...}}
```

## Mode 2: multi_agent_mode (CC=12)

**Flow**:
```
1. internal_rag_function(query)           → internal context
2. ainvoke_with_fallback(gap_brief_prompt) → LLM identifies what's missing
3. tavily_client.research(refined_brief)   → Tavily deep research endpoint
4. Poll every 10s until status="completed"
5. synthesize_results(internal + research_report) → final report
```

**Key difference from fast_mode**: Uses Tavily's full Research API (multi-step, multi-source, AI synthesis) instead of raw search. Higher quality, higher latency (~minutes).

**Gap identification prompt pattern**:
```python
refine_brief_prompt = f"""Context (internal research): {internal_results}
Identify what information is missing or incomplete above. Output a concise research
prompt for a web search to fill those gaps. Do not reference or include any of the
internal research content — just output the research prompt."""
```

## Structured Output Support
Both modes accept `output_schema: Optional[Type[OutputSchema]]`. When provided:
- In fast_mode: schema informs subquery generation
- In multi_agent_mode: `schema.to_tavily_schema()` passed to Tavily Research API

## Usage Metrics Collected
```python
{
    "response_time": float,            # end-to-end
    "internal_function_response_time": float,
    "tavily": {"total_credits": ..., "search_count": ..., "search_response_time": ...},
    "llm": {"total_input_tokens": ..., "total_output_tokens": ..., "llm_call_count": ...},
    # multi_agent_mode only:
    "tavily_research_response_time": float
}
```

## Complexity Note
`multi_agent_mode` has CC=12 (highest in the package). This is driven by the polling loop, structured output branching, and usage dict assembly. Worth splitting into smaller functions in a future refactor.

## Relations
- implements [[Hybrid Intelligence Pattern]]
- uses [[LLM Fallback Cascade Pattern]]
- uses [[LLM Provider Abstraction Pattern]]
- part_of [[agent-toolkit — README]]