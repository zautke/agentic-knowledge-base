---
title: ReAct Agent Loop Pattern
type: pattern
permalink: agent-kb/reference/patterns/re-act-agent-loop-pattern
created: 2026-04-18
modified: 2026-04-18
entity_type: pattern
status: evergreen
tags:
- project/agent-kb
- domain/agents
- domain/patterns
- domain/llm
- framework/anthropic-sdk
- framework/langgraph
---

# ReAct Agent Loop Pattern

## Summary

The Reason+Act (ReAct) loop is the canonical control structure for tool-using LLM agents. The model **observes** context, **reasons** about what to do next (which tool to call or whether to respond), **acts** by executing a tool, and **repeats** until it produces a terminal text response.

Three implementation tiers exist in `tavily-agent-toolkit` use-cases, ordered from most to least control:

| Tier | Implementation | Loop ownership |
|------|---------------|----------------|
| Manual | Raw Anthropic `messages.create()` + `while True:` | Caller-managed |
| SDK-managed | `claude_agent_sdk.ClaudeSDKClient` | SDK-managed |
| Framework | LangGraph `create_agent().ainvoke()` | Framework-managed |

---

## Core Pattern (Manual — Most Instructive)

The chatbot use-case (`use-cases/claude_sdk/chatbot.py:120`) demonstrates the raw loop:

```python
# Outer: conversation history accumulates across turns
messages = []
while True:
    user_input = input("You: ").strip()
    messages.append({"role": "user", "content": user_input})

    # Inner: agent loop — keep calling until no more tool calls
    while True:
        response = client.messages.create(
            model="claude-haiku-4-5-20251001",
            max_tokens=4096,
            system=SYSTEM_PROMPT,
            tools=TOOLS,
            messages=messages
        )

        if response.stop_reason == "tool_use":
            # OBSERVE: model requested tool use
            assistant_content = response.content
            messages.append({"role": "assistant", "content": assistant_content})

            # ACT: execute each tool and collect results
            tool_results = []
            for block in assistant_content:
                if block.type == "tool_use":
                    result = await execute_tool(block.name, block.input)
                    tool_results.append({
                        "type": "tool_result",
                        "tool_use_id": block.id,
                        "content": result
                    })

            messages.append({"role": "user", "content": tool_results})
            # REPEAT: model sees tool results, continues reasoning

        else:
            # TERMINAL: model produced final text response
            for block in response.content:
                if hasattr(block, "text"):
                    print(block.text)
            messages.append({"role": "assistant", "content": response.content})
            break  # Exit inner loop; resume outer conversation loop
```

**Key invariant**: Tool results are injected back as a `user` role message with `type: "tool_result"` items matching each `tool_use_id`. The model cannot proceed without them.

---

## SDK-Managed Variant

The company intelligence agent (`use-cases/claude_sdk/company_intelligence_deep_agent.py:192`) uses `claude_agent_sdk` which manages the multi-turn loop server-side:

```python
async with ClaudeSDKClient(options=agent_options) as client:
    await client.query(prompt)  # Submit initial prompt

    async for msg in client.receive_response():
        if isinstance(msg, AssistantMessage):
            for block in msg.content:
                if isinstance(block, ToolUseBlock):
                    # OBSERVE + LOG: model is using a tool
                    step += 1
                    label = TOOL_LABELS.get(block.name, block.name)
                    print(f"  [{step}] {label}")
                elif isinstance(block, TextBlock) and block.text:
                    # TERMINAL: final synthesis
                    report = block.text

        elif isinstance(msg, ResultMessage):
            if msg.result:
                report = msg.result
            # Session metadata: duration, turns, cost
            print(f"Completed in {msg.duration_ms/1000:.1f}s | {msg.num_turns} turns")
```

The SDK dispatches tool calls to registered MCP tool servers and feeds results back automatically. The caller only observes the event stream — no manual `messages.append()`.

---

## Framework-Managed Variant (LangGraph)

The LangGraph chatbot (`use-cases/langgraph/chatbot.py:18`) abstracts away the loop entirely:

```python
def create_chatbot_agent(model_config: ModelConfig):
    @tool
    async def search_and_format_tool(queries: list[str], time_range: str = None) -> str:
        return await search_and_format(queries=queries, api_key=TAVILY_API_KEY)

    return create_agent(
        model=model_config.model.model,
        tools=[search_and_format_tool, stream_research_tool],
        system_prompt="..."
    )

# Usage — loop entirely implicit:
response = await agent.ainvoke({"messages": messages})
assistant_message = response["messages"][-1].content
```

No explicit loop required; `ainvoke` runs the full ReAct cycle and returns when done.

---

## Tool Registry Pattern

The tool registry (from `TOOLS` constant, `chatbot.py:32`) is a JSON schema descriptor array that constrains what the model can call:

```python
TOOLS = [
    {
        "name": "search_and_format",
        "description": "Use for simple, single-fact lookups...",
        "input_schema": {
            "type": "object",
            "properties": {
                "queries": {"type": "array", "items": {"type": "string"}},
                "time_range": {"type": "string", "enum": ["day", "week", "month", "year"]}
            },
            "required": ["queries"]
        }
    },
    {
        "name": "stream_research",
        "description": "Use for multi-source, complex research...",
        "input_schema": {
            "type": "object",
            "properties": {"input": {"type": "string"}},
            "required": ["input"]
        }
    }
]
```

Tool descriptions are the primary signal for model selection — precision in description directly controls routing quality.

---

## System Prompt Role

The system prompt defines the agent's operating mode, tool-selection heuristics, and output constraints. From the company intelligence agent:

```
You are a business intelligence analyst researching companies.

You have three tools available:
- `crawl_company_website` – Crawl a company's website to discover and summarize pages
- `extract_from_urls` – Extract detailed content from specific URLs
- `tavily_search` – Search the web for news, funding, reviews, and other external info

Use these tools as you see fit to gather comprehensive information.
Combine website insights with external sources for a complete picture.

When you're done researching, write up your findings in a clear report.
Include citations [1], [2], etc. linking to your sources, and list all sources at the end.
```

**Pattern**: List each tool with its name + single-sentence description. Explicit guidance on *when* to stop (write report, cite sources) prevents infinite loops.

---

## Key Invariants

1. **Every `ToolUseBlock` must receive a `tool_result`** with matching `tool_use_id` — the model blocks otherwise.
2. **Stop reason drives the branch**: `"tool_use"` → execute+continue; any other → terminate inner loop.
3. **History accumulates**: Both user messages and assistant+tool messages append to `messages`. Full history is re-sent on every turn (stateless API).
4. **Outer vs inner loops**: Outer loop manages conversation turns; inner loop manages tool execution within one turn.
5. **SDK/framework variants trade control for brevity**: Manual = full observability; framework = maximum concision.

---

## Anti-Patterns

| Anti-Pattern | Problem |
|---|---|
| Calling `messages.create()` once without inner loop | Agent can only use one tool per user turn |
| Not appending tool results before next LLM call | Model hangs or produces hallucinated tool outcomes |
| No stop condition in inner loop | Infinite loop if `stop_reason` unexpected |
| Over-specifying tool descriptions | Model ignores or misroutes tools |
| Vague system prompt terminal condition | Model doesn't know when to stop calling tools |

---

## Relations

- Implements: [[Hybrid Intelligence Pattern]] — HybridResearcher uses multi_agent_mode which orchestrates sub-agents via this loop
- Used by: [[agent-toolkit — Company Intelligence Use Case]]
- Used by: [[agent-toolkit — Chatbot Use Case]]
- Related: [[LLM Fallback Cascade Pattern]] — ainvoke_with_fallback wraps individual LLM calls within the loop
- Extends: [[agent-toolkit — Context Engineering Pipeline]] — tool output runs through format_web_results before being appended

---

## Evolution Log

| Date | Change | Trigger |
|------|--------|---------|
| 2026-04-18 | Created from company_intelligence_deep_agent.py + chatbot.py source extraction | Phase 3-T3 ingestion |