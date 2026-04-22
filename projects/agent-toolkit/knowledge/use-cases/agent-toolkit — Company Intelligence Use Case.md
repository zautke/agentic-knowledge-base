---
title: agent-toolkit — Company Intelligence Use Case
type: example
permalink: agent-kb/projects/agent-toolkit/knowledge/use-cases/agent-toolkit-company-intelligence-use-case
created: 2026-04-18
modified: 2026-04-18
entity_type: example
status: evergreen
tags:
- project/agent-toolkit
- domain/agents
- domain/use-cases
- framework/anthropic-sdk
- framework/claude-agent-sdk
---

# Company Intelligence Use Case

## Source

`use-cases/claude_sdk/company_intelligence_deep_agent.py`

## Overview

A deep-research agent that produces comprehensive business intelligence reports on any company. Uses the `claude_agent_sdk` (MCP-over-subprocess) to run a multi-turn ReAct loop with three tools: website crawl, URL extraction, and web search. The agent autonomously decides how many tool calls to make.

**Output**: Markdown report with numbered citations and a sources section.

---

## Architecture

```
User input (company_name, website_url, focus)
    ↓
_build_prompt() → structured task string
    ↓
ClaudeSDKClient (claude_agent_sdk) — async event stream
    ↓ ToolUseBlock → crawl_company_website | extract_from_urls | tavily_search
    ↓ (loop until TextBlock / ResultMessage)
Final report string (Markdown with citations)
```

---

## Tool Roster

| Tool | Wraps | Purpose |
|------|-------|---------|
| `crawl_company_website` | `crawl_and_summarize()` | Full-site crawl with LLM summarization |
| `extract_from_urls` | `extract_and_summarize()` | Per-URL deep extraction |
| `tavily_search` | `search_dedup()` | Multi-query deduped web search |

All tools are registered as MCP tool servers and dispatched by the SDK, not the caller.

---

## Key Implementation Details

### Tool Coercion Guard

```python
async def tavily_search(args: dict[str, Any]) -> dict[str, Any]:
    # The model sometimes passes queries as a comma-separated string instead
    # of a list, and numeric params as strings — coerce to expected types.
    queries = args.get("queries") or []
    if isinstance(queries, str):
        queries = [q.strip() for q in queries.split(",") if q.strip()]
    max_results = int(args.get("max_results", 5))
```

**Why**: The model does not always respect JSON schema constraints — defensive coercion prevents runtime errors.

### Event Stream Loop

```python
async with ClaudeSDKClient(options=agent_options) as client:
    await client.query(prompt)
    async for msg in client.receive_response():
        if isinstance(msg, AssistantMessage):
            for block in msg.content:
                if isinstance(block, ToolUseBlock):
                    step += 1
                    label = TOOL_LABELS.get(block.name, block.name)
                    print(f"  [{step}] {label}{detail}")
                elif isinstance(block, TextBlock) and block.text:
                    report = block.text
        elif isinstance(msg, ResultMessage):
            if msg.result:
                report = msg.result
```

### Progress Display

```python
TOOL_LABELS: dict[str, str] = {
    "mcp__company-intel-tools__crawl_company_website": "Crawling website",
    "mcp__company-intel-tools__extract_from_urls": "Extracting URLs",
    "mcp__company-intel-tools__tavily_search": "Searching the web",
}
```

MCP tool names are fully qualified (`mcp__<server-name>__<tool>`). The `TOOL_LABELS` dict maps them to human-readable progress strings.

### System Prompt Design

The system prompt:
- Lists tools by name with one-sentence descriptions
- Instructs the model to combine website and external sources
- Specifies terminal condition: "write up your findings in a clear report"
- Mandates citation format: `[1], [2]` in-text + Sources list at end

---

## Usage

```python
from company_intelligence_deep_agent import run_research

report = await run_research(
    company_name="Tavily",
    website_url="https://tavily.com",
    focus="product offering, pricing, and competitive positioning"
)
print(report)
```

---

## Dependencies

- `claude_agent_sdk` — MCP-over-subprocess SDK (AssistantMessage, ToolUseBlock, TextBlock, ResultMessage, ClaudeAgentOptions)
- `tavily_agent_toolkit` — `crawl_and_summarize`, `extract_and_summarize`, `search_dedup`, `format_web_results`, `ModelConfig`

---

## Relations

- Implements: [[ReAct Agent Loop Pattern]]
- Uses: [[agent-toolkit — crawl_and_summarize Tool]]
- Uses: [[agent-toolkit — extract_and_summarize Tool]]
- Uses: [[agent-toolkit — async_search_and_dedup Tool]]
- Related: [[agent-toolkit — HybridResearcher Agent]]

---

## Evolution Log

| Date | Change | Trigger |
|------|--------|---------|
| 2026-04-18 | Created from jcodemunch extraction of company_intelligence_deep_agent.py | Phase 3-T4 ingestion |