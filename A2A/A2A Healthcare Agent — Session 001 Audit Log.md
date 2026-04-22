---
title: A2A Healthcare Agent — Session 001 Audit Log
type: note
permalink: kb/a2-a/a2-a-healthcare-agent-session-001-audit-log
tags:
- a2a
- session-history
- audit
- '2026-03-29'
---

# Session 001 — March 2026 SOTA Audit

**Date:** 2026-03-29
**Scope:** Full codebase audit + dependency SOTA research

## Files Read

- a2a_healthcare_agent.py (BeeAI orchestrator)
- a2a_policy_agent.py, policy_agent_executor.py (Policy A2A server)
- a2a_provider_agent.py (Provider A2A server)
- a2a_research_agent.py (Google ADK research agent)
- agents.py (PolicyAgent + ProviderAgent implementations)
- mcpserver.py (FastMCP doctor DB server)
- L3/agents.py (earlier lesson version)
- requirements.txt, pyproject.toml

## Searches Run

- A2A protocol latest → a2a-sdk 0.3.25 / 1.0.0a0 (Mar 2026)
- BeeAI Framework → 0.1.78 (Feb 2026)
- Google ADK → 0.6.0 GA vs project 1.19.0 (discrepancy)
- LangChain/LangGraph 1.0 → create_agent confirmed new API
- Gemini 2.5 → Flash + Pro GA on Vertex AI
- Claude Haiku 4.5 → current, Vertex string @20251001 valid
- langchain-mcp-adapters → 0.2.2

## Critical Finding

`gemini-3.1-pro-preview` in a2a_research_agent.py:26 is a NON-EXISTENT model ID.
Research agent will fail at startup with a Vertex API error.
Fix: change to `gemini-2.5-pro-preview`.

## Outputs Created

- PLANNING.md — strategic update plan with dependency table, code bugs, protocol gaps, 3-phase plan
- TASKS.md — 20+ prioritized tasks across CRITICAL/HIGH/MEDIUM/LOW/RESEARCH
- SESSION_HISTORY.md — this log
- basic-memory notes: A2A/planning, A2A/tasks, A2A/session-001

## Next Steps

Start with FIX-1 (gemini model ID fix), then run uv pip install upgrade pass for DEP-1 through DEP-3.
