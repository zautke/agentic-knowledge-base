---
title: A2A Healthcare Agent — TASKS tracker
type: note
permalink: kb/a2-a/a2-a-healthcare-agent-tasks-tracker
tags:
- a2a
- tasks
- todo
- modernization
---

# A2A Healthcare Agent — Modernization Tasks

> Generated: 2026-03-29 | Full detail in repo TASKS.md

## CRITICAL

- [ ] Fix `gemini-3.1-pro-preview` → `gemini-2.5-pro-preview` in `a2a_research_agent.py:26`
- [ ] Verify `langchain.agents.create_agent` exists in langchain==1.0.2
- [ ] Make provider model configurable via `PROVIDER_MODEL_ID` env var

## HIGH (dep upgrades)

- [ ] Bump a2a-sdk: 0.3.16 → 0.3.25
- [ ] Bump beeai-framework: 0.1.70 → 0.1.78
- [ ] Bump langchain-mcp-adapters: 0.1.11 → 0.2.2
- [ ] Investigate google-adk version discrepancy (1.19.0 vs 0.6.0 GA)
- [ ] Verify anthropic[vertex] model string for Claude Haiku 4.5

## MEDIUM (protocol + arch)

- [ ] Enable SSE streaming on Policy Agent (a2a_policy_agent.py)
- [ ] Enable SSE streaming on Provider Agent (a2a_provider_agent.py)
- [ ] Enable SSE streaming on Research Agent (ADK may handle automatically)
- [ ] Add pydantic-settings config module (settings.py)
- [ ] Replace UnconstrainedMemory → SlidingWindowMemory in healthcare orchestrator
- [ ] Add MCP_TRANSPORT env var to mcpserver.py
- [ ] Make LRU maxsize configurable via MAX_SESSIONS env var

## LOW (production hardening)

- [ ] Add OpenTelemetry tracing (BeeAI 0.1.75+ OTEL integration)
- [ ] Add agent card signing / authentication
- [ ] Replace InMemoryTaskStore → Redis-backed persistent store
- [ ] Write docker-compose.yml for 5-service orchestration
- [ ] Add /health endpoints to all agents
- [ ] Add tenacity retry logic for inter-agent A2A calls

## RESEARCH

- [ ] Evaluate a2a-sdk 1.0.0a0 alpha stability
- [ ] Evaluate gRPC transport for intra-cluster calls
- [ ] Evaluate gemini-2.5-flash-lite for research agent cost reduction
- [ ] Clarify agent-framework vs beeai-framework relationship
