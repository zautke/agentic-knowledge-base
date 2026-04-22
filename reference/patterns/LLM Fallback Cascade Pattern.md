---
title: LLM Fallback Cascade Pattern
type: pattern
permalink: agent-kb/reference/patterns/llm-fallback-cascade-pattern
created: 2026-04-18
modified: 2026-04-18
entity_type: pattern
status: evergreen
tags:
- domain/agent-engineering
- domain/llm-reliability
- domain/patterns
- status/evergreen
---

# LLM Fallback Cascade Pattern

## Agent Curation Metaprompt
```
ROLE: You are the curator for this global reference pattern.
Accumulate evidence and examples. Never summarize away specifics.
Update the Examples section when new implementations are observed.
```

## Summary
A reliability pattern where an LLM invocation is backed by an ordered cascade of fallback models. Each model in the cascade gets exactly 1 attempt (with 1 retry if no fallbacks configured). If all models fail, the last exception is re-raised.

## Problem
LLM API calls fail — timeouts, rate limits, model deprecations, provider outages. Without a fallback strategy, a single failure breaks the entire agent.

## Solution
Configure a `ModelConfig` with a primary model and ordered fallback list. Pass this to `ainvoke_with_fallback`, which:
1. Tries primary model (1 attempt if fallbacks exist, 2 if not)
2. On failure, cascades to next model
3. Returns `LLMResponse(result, usage)` on success
4. Raises last exception if all models fail

## Reference Implementation
**Source**: `utilities/utils.py::ainvoke_with_fallback` — `local/agent-toolkit-49aa8c6c`

```python
async def ainvoke_with_fallback(
    model_config: ModelConfig,
    messages: Union[list[dict], str],
    output_schema: Optional[Type[BaseModel]] = None,
    return_usage: bool = False,
    **invoke_kwargs: Any
) -> Any:
    all_models = model_config.get_all_models()
    has_fallbacks = len(all_models) > 1
    last_error: Optional[Exception] = None
    
    for i, model_obj in enumerate(all_models):
        max_attempts = 1 if has_fallbacks else 2
        for attempt in range(max_attempts):
            try:
                llm = init_chat_model(**model_config.to_init_kwargs(model_obj if i > 0 else None))
                # ... invoke with optional structured output
                # ... return LLMResponse if return_usage=True
            except Exception as e:
                last_error = e
                if attempt < max_attempts - 1:
                    await asyncio.sleep(2 ** attempt)
    raise last_error
```

## Configuration Pattern
```python
from langchain.chat_models import init_chat_model  # 20+ providers

config = ModelConfig(
    model=ModelObject(model="gpt-5", api_key="..."),
    fallback_models=[
        ModelObject(model="claude-sonnet-4-5-20250514"),
        ModelObject(model="gemini-2.5-flash"),
    ],
    temperature=0.7
)
```

## Key Invariants
- **1 attempt per model** when fallbacks configured (never retry the same model if a cheaper fallback exists)
- **2 attempts for primary** when no fallbacks (standard single-model retry)
- **Exponential backoff** between attempts: `sleep(2^attempt)` seconds
- **`include_usage=True` always forced** in retry wrappers — observability is non-negotiable
- Structured output (Pydantic) supported via `with_structured_output(schema, include_raw=True)`

## Usage Tracking Integration
Returns `LLMResponse(result=..., usage=LLMUsage(...))` when `return_usage=True`. All callers in `fast_mode` and `multi_agent_mode` pass `return_usage=True` and call `usage.llm.merge(response.usage)`.

## Known in These Repos
- [2026-04-18] `tavily-agent-toolkit` v0.1.0 — primary implementation — `agents/`, `tools/`

## Evolution Log

| Date | Change | Reason | Trigger |
|---|---|---|---|
| 2026-04-18 | Initial pattern documented | Extracted from `ainvoke_with_fallback` during agent-toolkit KB seeding | Global truth promotion — Bridge Protocol |

## Relations
- derived_from [[agent-toolkit — README]]
- related_to [[LLM Provider Abstraction Pattern]]
- implements [[Curator Handover Knowledge]]