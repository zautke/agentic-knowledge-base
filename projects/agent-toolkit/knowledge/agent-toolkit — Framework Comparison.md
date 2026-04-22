---
title: agent-toolkit — Framework Comparison
type: reference
permalink: agent-kb/projects/agent-toolkit/knowledge/agent-toolkit-framework-comparison
created: 2026-04-18
modified: 2026-04-18
entity_type: reference
status: evergreen
tags:
- project/agent-toolkit
- domain/agents
- domain/use-cases
- framework/anthropic-sdk
- framework/langgraph
- framework/claude-agent-sdk
---

# Framework Comparison: SDK vs LangGraph vs Claude Agent SDK

## Scope

Three frameworks appear in `tavily-agent-toolkit` use-cases. This note compares them across implementation dimensions using the chatbot and company intelligence use-cases as grounding.

---

## Comparison Matrix

| Dimension | Raw Anthropic SDK | claude_agent_sdk | LangGraph |
|-----------|-------------------|-----------------|-----------|
| **Use case** | Chatbot (claude_sdk/chatbot.py) | Company Intelligence (claude_sdk/company_intelligence_deep_agent.py) | Chatbot (langgraph/chatbot.py) |
| **Loop control** | Caller: `while True: if stop_reason == tool_use` | SDK: event stream `async for msg` | Framework: `agent.ainvoke()` |
| **Tool dispatch** | Manual `execute_tool()` | MCP server subprocess | LangGraph tool nodes |
| **Tool registration** | JSON schema dict in `TOOLS` constant | MCP server config at session init | `@tool` decorator |
| **Message history** | Manual `messages.append()` | SDK-internal | Framework-managed |
| **Model binding** | Hardcoded string literal | `ClaudeAgentOptions(model=...)` | `ModelConfig.model.model` |
| **Provider lock** | Anthropic only | Anthropic only (via claude_agent_sdk) | Provider-agnostic (via `init_chat_model`) |
| **Observability** | Full: can log any block type | Event stream: `ToolUseBlock`, `TextBlock`, `ResultMessage` (duration, cost, turns) | Limited without custom callbacks |
| **Lines of loop code** | ~30 lines | ~15 lines (no dispatch) | ~3 lines |
| **Type safety** | Low (dict blocks) | Medium (typed message classes) | Medium (`@tool` type hints) |

---

## When to Use Each

### Raw Anthropic SDK
- **When**: You need full control over message history, tool dispatch timing, or streaming; prototyping; learning the API
- **Cost**: Verbose; you own all error handling
- **Example**: Interactive chatbot where you need fine-grained control over context

### claude_agent_sdk (MCP-based)
- **When**: Tools are standalone microservices (MCP servers); you want cost/duration telemetry (`ResultMessage`); multi-step deep research with many tool calls
- **Cost**: Requires MCP server setup; Anthropic-only
- **Example**: Company intelligence — 10+ tool calls across crawl + search + extract

### LangGraph
- **When**: Multi-provider support needed; team familiar with LangChain ecosystem; want framework handles retry/routing logic
- **Cost**: Framework abstraction hides what the loop is doing; debugging requires LangGraph knowledge
- **Example**: Chatbot with swappable models via `ModelConfig`

---

## Tool Description Parity

Both chatbot implementations use the same two-tier routing heuristic via tool descriptions:

```
search_and_format → "simple, straightforward questions, single-fact lookups"
stream_research   → "multiple sources, detailed analysis, anything not trivially simple"
```

The LangGraph system prompt adds a hard constraint absent from the SDK version:
```
CRITICAL: YOU CAN ONLY USE THE RESEARCH TOOL ONCE.
```
This prevents the LangGraph agent from issuing parallel research calls (expensive). The SDK version relies on the model's judgment.

---

## Divergent Patterns

| Pattern | SDK chatbot.py | LangGraph chatbot.py |
|---------|---------------|----------------------|
| Tool functions | Standalone async functions | `@tool`-decorated methods inside factory |
| Agent factory | Not used | `create_chatbot_agent(model_config)` |
| Model selection | String literal | `ModelConfig.model.model` |
| Research result | `json.dumps({"route": ..., "response": ...})` string | `{"route": ..., "response": ...}` dict |

---

## Relations

- Used by: [[agent-toolkit — Chatbot Use Case]]
- Related: [[LLM Provider Abstraction Pattern]] — LangGraph variant uses ModelConfig; SDK variant does not
- Related: [[ReAct Agent Loop Pattern]] — both implement the same loop at different abstraction levels

---

## Evolution Log

| Date | Change | Trigger |
|------|--------|---------|
| 2026-04-18 | Created from get_symbol_diff analysis of both chatbot.py files | Phase 3-T4 ingestion |