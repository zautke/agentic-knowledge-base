---
title: agent-toolkit — Subquery Generation Pattern
type: pattern
permalink: agent-kb/projects/agent-toolkit/knowledge/patterns/agent-toolkit-subquery-generation-pattern
created: 2026-04-18
modified: 2026-04-18
entity_type: pattern
status: evergreen
tags:
- project/agent-toolkit
- domain/context-engineering
- domain/patterns
- domain/llm
---

# Subquery Generation Pattern

## Source

`utilities/utils.py::generate_subqueries` (line 713)

## Summary

Uses an LLM to decompose a broad query into up to N focused, Google-style search queries that each cover a different subtopic. The subqueries are then fanned out to parallel web searches. This is the bridge between a user's high-level intent and the specific search terms needed to retrieve comprehensive, non-overlapping evidence.

Used by: `HybridResearcher.multi_agent_mode` and `search_and_answer`.

---

## Full Source

```python
async def generate_subqueries(
    query: str,
    model_config: ModelConfig,
    max_number_of_subqueries: int = 4,
    context: str | None = None,
    output_schema: Optional[Type[BaseModel]] = None,
    return_usage: bool = False,
) -> list[str] | tuple[list[str], LLMUsage]:
    """Generate subqueries to cover different subtopics of the main query."""
    prompt = f"""Generate up to {max_number_of_subqueries} short and directed Google-style
search queries covering different subtopics to answer: {query}
Only generate as many queries as needed to cover the topic comprehensively.
Do not include dates or years in the queries unless explicitly specified in the original query."""

    messages: list[dict] = [{"role": "system", "content": prompt}]

    if context:
        messages.append({
            "role": "user",
            "content": f"Here is the context that you can use to generate the subqueries: {context}\n"
                       f"We want to generate subqueries that fill the gaps in the context for the query: {query}"
        })
    if output_schema:
        messages.append({
            "role": "user",
            "content": f"Your research goal is to fill out this schema: {output_schema.model_json_schema()}"
        })

    if return_usage:
        response = await ainvoke_with_fallback(model_config, messages, output_schema=SubqueriesOutput, return_usage=True)
        subqueries = cast(SubqueriesOutput, response.result).subqueries
        return subqueries, response.usage
    else:
        result = await ainvoke_with_fallback(model_config, messages, output_schema=SubqueriesOutput)
        return cast(SubqueriesOutput, result).subqueries
```

---

## Design Decisions

### Structured Output (`SubqueriesOutput`)

Output is constrained to a `SubqueriesOutput` Pydantic model (`output_schema=SubqueriesOutput` passed to `ainvoke_with_fallback`). This guarantees a `list[str]` — no parsing required.

### Gap-Aware Context Injection

When `context` is provided (existing KB content), the prompt pivots from "cover the topic" to "fill the gaps in the context." This is the mechanism that makes the [[Data Enrichment Flywheel Pattern]] gap-aware: the model knows what's already known and only generates queries for what's missing.

### "Only generate as many as needed"

The prompt instructs the model to generate *up to* `max_number_of_subqueries` — not exactly N. This prevents unnecessary searches on narrow queries.

### No Dates Unless Specified

`"Do not include dates or years unless explicitly specified"` prevents the model from hallucinating temporal constraints that would exclude relevant results.

### Usage Tracking

`return_usage=True` → returns `(list[str], LLMUsage)` tuple. Plugs into the observability model (`LLMUsage` captures tokens, cost). This is consistent with the pattern across all LLM-calling utilities.

---

## Calling Pattern

```python
# Basic
subqueries = await generate_subqueries(
    query="What are the latest AI agent frameworks?",
    model_config=model_config,
    max_number_of_subqueries=4
)
# → ["best AI agent frameworks 2024", "LangGraph vs CrewAI comparison", ...]

# Gap-filling (with existing context)
subqueries = await generate_subqueries(
    query="What are the latest AI agent frameworks?",
    model_config=model_config,
    context=existing_kb_content,  # model generates queries for what's missing
    max_number_of_subqueries=4
)

# With usage tracking
subqueries, usage = await generate_subqueries(
    query="...",
    model_config=model_config,
    return_usage=True
)
```

---

## Integration Points

- **`search_and_answer`**: Calls `generate_subqueries` → fans out to `search_dedup` → formats results
- **`HybridResearcher.multi_agent_mode`**: Calls `generate_subqueries` with `context=internal_rag_result` to fill knowledge gaps
- **`ainvoke_with_fallback`**: All LLM calls go through this — fallback cascade applies

---

## Relations

- Implements: [[Data Enrichment Flywheel Pattern]] — gap-aware mode is the key mechanism
- Uses: [[LLM Fallback Cascade Pattern]] — via ainvoke_with_fallback
- Used by: [[agent-toolkit — search_and_answer Tool]]
- Used by: [[agent-toolkit — HybridResearcher Agent]]
- Related: [[agent-toolkit — Observability Data Model]] — return_usage feeds LLMUsage

---

## Evolution Log

| Date | Change | Trigger |
|------|--------|---------|
| 2026-04-18 | Created from jcodemunch symbol extraction of generate_subqueries | Phase 3-T3 ingestion |