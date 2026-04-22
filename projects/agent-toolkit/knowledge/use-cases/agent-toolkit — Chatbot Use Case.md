---
title: agent-toolkit — Chatbot Use Case
type: example
permalink: agent-kb/projects/agent-toolkit/knowledge/use-cases/agent-toolkit-chatbot-use-case
created: 2026-04-18
modified: 2026-04-18
entity_type: example
status: evergreen
tags:
- project/agent-toolkit
- domain/agents
- domain/use-cases
- framework/anthropic-sdk
- framework/langgraph
---

# Chatbot Use Case

## Two Implementations

The chatbot blueprint exists in two parallel implementations:

| File | Framework | Loop ownership | Model |
|------|-----------|----------------|-------|
| `use-cases/claude_sdk/chatbot.py` | Raw Anthropic SDK | Caller-managed `while True:` | `claude-haiku-4-5-20251001` |
| `use-cases/langgraph/chatbot.py` | LangGraph `create_agent` | Framework-managed | `gpt-5.1` via `ModelConfig` |

Both expose the same two tools: `search_and_format` (simple queries) and `stream_research` (complex research).

---

## Anthropic SDK Implementation

### Tool Routing Logic (Two-Tool Design)

```python
TOOLS = [
    {
        "name": "search_and_format",
        "description": "Use for very simple, straightforward questions that need minimal data
            (e.g., 'What is the capital of France?', single-fact lookups)",
        ...
    },
    {
        "name": "stream_research",
        "description": "Use for anything requiring multiple sources, detailed analysis,
            comparisons, comprehensive answers, or any query that isn't trivially simple.",
        ...
    }
]
```

**Design**: Tool descriptions encode the routing heuristic. The model reads descriptions to decide which to call — `search_and_format` for speed, `stream_research` for depth.

### Manual ReAct Loop

```python
while True:  # Outer: conversation turns
    user_input = input("You: ").strip()
    messages.append({"role": "user", "content": user_input})

    while True:  # Inner: agent execution within one turn
        response = client.messages.create(
            model="claude-haiku-4-5-20251001",
            tools=TOOLS,
            messages=messages
        )

        if response.stop_reason == "tool_use":
            messages.append({"role": "assistant", "content": response.content})
            tool_results = []
            for block in response.content:
                if block.type == "tool_use":
                    result = await execute_tool(block.name, block.input)
                    tool_results.append({
                        "type": "tool_result",
                        "tool_use_id": block.id,
                        "content": result
                    })
            messages.append({"role": "user", "content": tool_results})
        else:
            # Final response
            break
```

### Tool Execution

```python
async def execute_tool(tool_name: str, tool_input: dict) -> str:
    if tool_name == "search_and_format":
        return await search_and_format(
            queries=tool_input.get("queries", []),
            api_key=TAVILY_API_KEY,
            time_range=tool_input.get("time_range")
        )
    elif tool_name == "stream_research":
        client = TavilyClient(api_key=TAVILY_API_KEY)
        response = client.research(input=tool_input["input"], model="mini", max_results=10, stream=True)
        report = handle_research_stream(response, stream_content_generation=False)
        return json.dumps({"route": "research", "response": report})
```

`stream_research` calls Tavily's native research API and wraps the result in a JSON envelope for the model.

---

## LangGraph Implementation

### Agent Definition

```python
def create_chatbot_agent(model_config: ModelConfig):
    @tool
    async def search_and_format_tool(queries: list[str], time_range: str = None) -> str:
        """Search the web and return formatted results..."""
        return await search_and_format(queries=queries, api_key=TAVILY_API_KEY, time_range=time_range)

    @tool
    async def stream_research_tool(input: str) -> dict:
        """Research the web with AI synthesis..."""
        client = TavilyClient(api_key=TAVILY_API_KEY)
        response = client.research(input=input, model="mini", max_results=10, stream=True)
        report = handle_research_stream(response, stream_content_generation=False)
        return {"route": "research", "response": report}

    return create_agent(
        model=model_config.model.model,
        tools=[search_and_format_tool, stream_research_tool],
        system_prompt="...CRITICAL: YOU CAN ONLY USE THE RESEARCH TOOL ONCE..."
    )
```

**Notable**: System prompt includes hard constraint `"CRITICAL: YOU CAN ONLY USE THE RESEARCH TOOL ONCE"` — prevents the model from issuing expensive parallel research calls.

### Usage

```python
agent = create_chatbot_agent(config)
messages.append({"role": "user", "content": query})
response = await agent.ainvoke({"messages": messages})
assistant_message = response["messages"][-1].content
```

No tool dispatch code required — LangGraph handles the entire inner loop.

---

## Framework Comparison Summary

| Dimension | Anthropic SDK (chatbot.py) | LangGraph (chatbot.py) |
|-----------|--------------------------|------------------------|
| Loop code | ~30 lines, explicit | 0 lines, implicit |
| Tool registration | JSON schema dict | `@tool` decorator |
| Message history | Manual `.append()` | Framework-managed |
| Model binding | Hardcoded string | `ModelConfig.model.model` |
| Observability | Full (log any block) | Limited without hooks |
| Portability | Anthropic-specific | Provider-agnostic via `ModelConfig` |
| Complexity cost | Caller must understand API | Must understand LangGraph abstraction |

---

## Citations in System Prompt

Both implementations instruct the model to cite sources inline `[1], [2]` and list them at the end. This is a consistent pattern across all tavily-agent-toolkit use-cases.

---

## Relations

- Implements: [[ReAct Agent Loop Pattern]]
- Uses: [[agent-toolkit — search_and_format Tool]]
- Framework variant relates to: [[agent-toolkit — Framework Comparison]]
- Related: [[agent-toolkit — Company Intelligence Use Case]]

---

## Evolution Log

| Date | Change | Trigger |
|------|--------|---------|
| 2026-04-18 | Created from jcodemunch extraction of both chatbot.py files | Phase 3-T4 ingestion |