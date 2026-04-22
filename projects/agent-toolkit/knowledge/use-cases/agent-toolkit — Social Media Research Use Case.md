---
title: agent-toolkit — Social Media Research Use Case
type: example
permalink: agent-kb/projects/agent-toolkit/knowledge/use-cases/agent-toolkit-social-media-research-use-case
created: 2026-04-18
modified: 2026-04-18
entity_type: example
status: evergreen
tags:
- project/agent-toolkit
- domain/agents
- domain/use-cases
- framework/anthropic-sdk
---

# Social Media Research Use Case

## Source

`use-cases/claude_sdk/social_media_research.py`

## Overview

A single-tool research agent that searches social media platforms (TikTok, Reddit, Instagram, X, Facebook, LinkedIn) to synthesize public opinion, trend data, and platform-specific discourse on any topic. Uses the raw Anthropic SDK with a manual ReAct loop.

**Output**: Markdown report with platform-specific insights, inline citations, and source URLs.

---

## Single-Tool Design

Unlike the chatbot (2 tools) and company intelligence (3 tools), this agent exposes exactly one tool:

```python
TOOLS = [
    {
        "name": "search_social_media",
        "description": """Search social media platforms for discussions, opinions, and content.
Use this tool to search across TikTok, Reddit, Instagram, X, Facebook, and LinkedIn.
You can search a specific platform or all platforms at once with "combined".""",
        "input_schema": {
            "type": "object",
            "properties": {
                "query": {"type": "string"},
                "platform": {
                    "type": "string",
                    "enum": ["tiktok", "facebook", "instagram", "reddit", "linkedin", "x", "combined"]
                },
                "max_results": {"type": "integer"},
                "include_raw_content": {"type": "boolean"},
                "time_range": {"type": "string", "enum": ["day", "week", "month", "year"]}
            },
            "required": ["query"]
        }
    }
]
```

The model calls this tool multiple times with different `platform` values or queries to build a multi-platform picture.

---

## System Prompt: Platform Routing Guidance

```
You research topics by searching social media platforms.

Search multiple times with different queries or platforms to get a complete picture.
Reddit is great for honest opinions, TikTok for trends, X for real-time reactions,
LinkedIn for professional takes.

Synthesize what you find into a clear answer. Include inline citations [1], [2]
and list sources with URLs at the end.
```

**Pattern**: System prompt contains platform-selection heuristics ("Reddit for opinions, TikTok for trends") so the model makes intelligent routing decisions without requiring the user to specify platforms.

---

## Tool Execution

```python
def execute_tool(tool_name: str, tool_input: dict) -> str:
    if tool_name == "search_social_media":
        result = social_media_search(
            query=tool_input.get("query", ""),
            api_key=TAVILY_API_KEY,
            platform=tool_input.get("platform", "combined"),
            include_raw_content=tool_input.get("include_raw_content", True),
            max_results=tool_input.get("max_results", 10),
            search_depth="advanced",
            include_answer=True,
            time_range=tool_input.get("time_range", "month"),
        )
        return json.dumps(result)
```

Note: `execute_tool` is **synchronous** here (no `async def`), unlike the chatbot variant. `social_media_search` is a sync wrapper.

---

## Research Loop

Standard ReAct loop (same as chatbot SDK variant):

```python
async def run_research(query: str) -> str:
    client = Anthropic(api_key=ANTHROPIC_API_KEY)
    messages = [{"role": "user", "content": query}]

    while True:
        response = client.messages.create(
            model="claude-haiku-4-5-20251001",
            max_tokens=4096,
            system=SYSTEM_PROMPT,
            tools=TOOLS,
            messages=messages
        )
        if response.stop_reason == "tool_use":
            messages.append({"role": "assistant", "content": response.content})
            tool_results = []
            for block in response.content:
                if block.type == "tool_use":
                    print(f"  [Searching social media on {block.input.get('platform', 'all')}...] ")
                    result = execute_tool(block.name, block.input)
                    tool_results.append({"type": "tool_result", "tool_use_id": block.id, "content": result})
            messages.append({"role": "user", "content": tool_results})
        else:
            for block in response.content:
                if hasattr(block, "text"):
                    return block.text
            return ""
```

---

## Platform Coverage

| Platform | Best For | Enum value |
|----------|----------|------------|
| Reddit | Honest opinions, niche communities | `reddit` |
| TikTok | Viral trends, Gen-Z discourse | `tiktok` |
| X (Twitter) | Real-time reactions, breaking news | `x` |
| LinkedIn | Professional perspectives, B2B | `linkedin` |
| Instagram | Visual content, influencer trends | `instagram` |
| Facebook | Mainstream consumer sentiment | `facebook` |
| Combined | All platforms simultaneously | `combined` |

---

## Relations

- Implements: [[ReAct Agent Loop Pattern]]
- Uses: [[agent-toolkit — social_media_search Tool]]
- Related: [[agent-toolkit — Chatbot Use Case]] — same SDK, different tool roster
- Related: [[agent-toolkit — Company Intelligence Use Case]] — same research pattern, different domain

---

## Evolution Log

| Date | Change | Trigger |
|------|--------|---------|
| 2026-04-18 | Created from jcodemunch extraction of social_media_research.py | Phase 3-T4 ingestion |