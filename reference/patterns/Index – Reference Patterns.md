---
title: Index – Reference Patterns
type: index
permalink: agent-kb/reference/patterns/index-reference-patterns
created: 2026-04-18
modified: 2026-04-18
entity_type: index
status: evergreen
tags:
- project/agent-kb
- domain/patterns
- domain/agents
- domain/llm
---

# Index – Reference Patterns

## Purpose

Navigation hub for global, reusable patterns that apply across projects. These are extracted from production implementations and promoted here because they describe general truths — not project-specific details.

**Bridge Protocol rule**: If a pattern applies to multiple systems or could be reused by any agent, it lives here, not inside a project folder.

---

## LLM and Provider Patterns

- [[LLM Fallback Cascade Pattern]] — `ainvoke_with_fallback`: cascade through fallback models, 1-attempt-per-model when fallbacks configured, 2 without
- [[LLM Provider Abstraction Pattern]] — `ModelConfig`/`ModelObject`: provider-agnostic config supporting 20+ LLM providers via `init_chat_model`

---

## Agent Architecture Patterns

- [[ReAct Agent Loop Pattern]] — Observe→Reason→Act: manual (Anthropic SDK), SDK-managed (claude_agent_sdk), framework-managed (LangGraph) variants
- [[Hybrid Intelligence Pattern]] — Internal RAG + real-time web search synthesis; fast mode (single-shot) and multi-agent mode (deep research)

---

## Data and Knowledge Patterns

- [[Data Enrichment Flywheel Pattern]] — Self-reinforcing loop: web findings → KB → gap-aware future queries; compounds over time

---

## Index Coverage

| Pattern | Origin | Promoted from |
|---------|--------|--------------|
| LLM Fallback Cascade | tavily-agent-toolkit | utilities/utils.py::ainvoke_with_fallback |
| LLM Provider Abstraction | tavily-agent-toolkit | models.py::ModelConfig |
| ReAct Agent Loop | tavily-agent-toolkit | use-cases/claude_sdk/ |
| Hybrid Intelligence | tavily-agent-toolkit | agents/hybrid_researcher.py |
| Data Enrichment Flywheel | tavily-agent-toolkit | agents/hybrid_researcher.py + generate_subqueries |

---

## Related Indices

- [[Index – Agent Runtime and Capability Stack]] — runtime/protocol/gateway patterns
- [[agent-toolkit — Architecture Index]] — project-specific architecture notes linking back to these globals
- [[Index – Agent KB]] — root index

---

## Evolution Log

| Date | Change | Trigger |
|------|--------|---------|
| 2026-04-18 | Created; seeded with 5 global patterns extracted from tavily-agent-toolkit | Phase 5 hub update after agent-toolkit KB ingestion |