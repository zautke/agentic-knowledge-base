---
title: A2A Healthcare Agent — March 2026 PLANNING
type: note
permalink: kb/a2-a/a2-a-healthcare-agent-march-2026-planning
tags:
- a2a
- healthcare
- planning
- beeai
- google-adk
- langchain
- modernization
- '2026'
---

# A2A Healthcare Agent — March 2026 Modernization Plan

> Audit date: 2026-03-29 | Auditor: Claude Sonnet 4.6

## Architecture

Multi-agent healthcare concierge (DLAI/IBM course project):
- Healthcare Agent (BeeAI RequirementAgent, orchestrator)
- Policy Agent (a2a-sdk + Claude Haiku 4.5 via Vertex AI)
- Provider Agent (a2a-sdk + LangChain create_agent + FastMCP stdio)
- Research Agent (Google ADK LlmAgent + google_search)

All inter-agent comms use HTTP/JSON-RPC over A2A protocol.

## Dependency Gaps (pinned vs SOTA)

| Package | Pinned | SOTA Mar 2026 | Action |
|---------|--------|---------------|--------|
| a2a-sdk | 0.3.16 | 0.3.25 / 1.0.0a0 | UPGRADE |
| beeai-framework | 0.1.70 | 0.1.78 | UPGRADE |
| google-adk | 1.19.0 | 0.6.0 GA | INVESTIGATE |
| langchain-mcp-adapters | 0.1.11 | 0.2.2 | UPGRADE |
| anthropic[vertex] | 0.72.0 | latest | CHECK |

## Critical Bugs

1. **a2a_research_agent.py:26** — `gemini-3.1-pro-preview` does NOT exist → use `gemini-2.5-pro-preview`
2. **agents.py:89** — `openai/gpt-oss-20b-maas` is IBM-internal, not portable
3. **agents.py:17** — verify `langchain.agents.create_agent` exists in langchain 1.0.2

## Protocol Gaps vs A2A 0.3.x SOTA

- All agents use `streaming=False` — stable SSE streaming now available
- No agent card signing / authentication
- `InMemoryTaskStore` is ephemeral — not production-safe
- No gRPC transport (optional, but available in 0.3.25)

## Implementation Phases

- **Phase 1 (Day 1, 2h):** Fix gemini model ID, bump a2a-sdk + beeai-framework + langchain-mcp-adapters, make provider model configurable
- **Phase 2 (Day 1-2, 4h):** Enable streaming, add Pydantic settings, SlidingWindowMemory, MCP SSE transport
- **Phase 3 (Day 2-3, 4h):** OpenTelemetry, agent auth, Redis task store, docker-compose, health endpoints

## Model Strategy

| Role | Current | Recommended |
|------|---------|-------------|
| Orchestrator | gemini-2.5-flash ✓ | keep |
| Policy | claude-haiku-4-5@20251001 ✓ | keep |
| Research | gemini-3.1-pro-preview ❌ | gemini-2.5-pro-preview |
| Provider | openai/gpt-oss-20b-maas ⚠️ | env var / gemini-2.5-flash |

## Open Questions

1. `agent-framework==1.0.0b251120` vs `beeai-framework` — same package under IBM-internal name?
2. `google-adk 1.19.0` vs `0.6.0` GA — version discrepancy unexplained
3. Production target? GKE / Cloud Run deployment changes PROD phase priority
