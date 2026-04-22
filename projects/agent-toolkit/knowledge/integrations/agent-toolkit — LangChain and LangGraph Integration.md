---
title: agent-toolkit — LangChain and LangGraph Integration
type: reference
permalink: agent-kb/projects/agent-toolkit/knowledge/integrations/agent-toolkit-lang-chain-and-lang-graph-integration
created: 2026-04-18
modified: 2026-04-18
entity_type: reference
status: evergreen
tags:
- project/agent-toolkit
- domain/integrations
- framework/langchain
- framework/langgraph
---

# LangChain and LangGraph Integration

## Overview

`tavily-agent-toolkit` integrates with LangChain/LangGraph at two layers:

1. **Provider abstraction** (`models.py`): `ModelConfig` calls `init_chat_model` from `langchain_core` to instantiate any of 20+ LLM providers
2. **Agent framework** (`use-cases/langgraph/`): LangGraph `create_agent` + `@tool` decorator replace the manual ReAct loop

---

## Layer 1: `init_chat_model` as the Provider Bridge

`ModelConfig.to_init_kwargs()` produces the parameter dict for LangChain's `init_chat_model`:

```python
from langchain.chat_models import init_chat_model

llm = init_chat_model(**model_config.to_init_kwargs(model_obj))
# Equivalent to:
# llm = init_chat_model(model="claude-haiku-4-5", model_provider="anthropic", api_key="sk-ant-...")
```

This is called inside `ainvoke_with_fallback` on every LLM invocation. The result is a LangChain `BaseChatModel`, which supports:
- `.ainvoke(messages)` for async calls
- `.with_structured_output(schema)` for Pydantic output constraints

See [[LLM Provider Abstraction Pattern]] for the full provider list and `ModelConfig` internals.

---

## Layer 2: LangGraph `create_agent`

The LangGraph chatbot uses LangGraph's `create_agent` factory:

```python
from langchain.agents import create_agent
from langchain_core.tools import tool

def create_chatbot_agent(model_config: ModelConfig):
    @tool
    async def search_and_format_tool(queries: list[str], time_range: str = None) -> str:
        """Tool docstring becomes the description passed to the model."""
        return await search_and_format(queries=queries, api_key=TAVILY_API_KEY)

    @tool
    async def stream_research_tool(input: str) -> dict:
        """Research the web with AI synthesis."""
        ...

    return create_agent(
        model=model_config.model.model,  # string model ID
        tools=[search_and_format_tool, stream_research_tool],
        system_prompt="..."
    )

# Usage:
agent = create_chatbot_agent(config)
response = await agent.ainvoke({"messages": messages})
assistant_message = response["messages"][-1].content
```

### `@tool` Decorator

LangChain's `@tool` decorator:
- Extracts the function's docstring as the tool description
- Infers the JSON schema from Python type annotations
- No separate `input_schema` dict needed (unlike raw Anthropic SDK)

### StateGraph Under the Hood

`create_agent` returns a compiled `StateGraph` (LangGraph). The `.ainvoke()` call runs:
1. Model node (LLM produces response)
2. Tool node (if `tool_use` detected)
3. Model node again (processes tool results)
4. Repeat until terminal

The `messages` key in the state dict accumulates history automatically.

---

## Key Difference from Raw SDK

| Dimension | Raw Anthropic SDK | LangGraph |
|-----------|------------------|-----------|
| Tool schema | `input_schema` JSON dict | Type annotations + docstring |
| Loop code | `while True: if stop_reason` | Implicit in `ainvoke` |
| Model binding | String literal | `ModelConfig.model.model` |
| Provider | Anthropic only | Any `init_chat_model` provider |

---

## LangGraph Social Media Research

`use-cases/langgraph/social_media_research.py` follows the same pattern as the chatbot — `@tool`-decorated `social_media_search_tool`, factory function, `agent.ainvoke()`.

---

## Dependencies

```toml
# agent-toolkit/pyproject.toml
dependencies = [
    "langchain",
    "langchain-core",
    "langgraph",
    ...
]
```

LangChain/LangGraph are hard dependencies. `init_chat_model` is the integration point used by all LLM calls.

---

## Relations

- Implements: [[LLM Provider Abstraction Pattern]] — via init_chat_model
- Related: [[ReAct Agent Loop Pattern]] — LangGraph abstracts the loop
- Related: [[agent-toolkit — Framework Comparison]]
- Used by: [[agent-toolkit — Chatbot Use Case]] (LangGraph variant)

---

## Evolution Log

| Date | Change | Trigger |
|------|--------|---------|
| 2026-04-18 | Created from LangGraph chatbot source + ModelConfig analysis | Phase 3-T5 ingestion |