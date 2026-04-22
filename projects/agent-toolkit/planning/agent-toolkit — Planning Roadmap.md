---
title: agent-toolkit — Planning Roadmap
type: reference
permalink: agent-kb/projects/agent-toolkit/planning/agent-toolkit-planning-roadmap
created: 2026-04-18
modified: 2026-04-18
entity_type: reference
status: evergreen
tags:
- project/agent-toolkit
- domain/curation
- op/planning
- status/evergreen
---

# Agent Toolkit KB — Ingestion Roadmap

## Tier 1: Core Architecture (Foundation)
- [x] Project scaffold (project.json, README, CONTRIBUTING, architecture, planning, sessions, ops, scratch, meta)
- [x] Gated KB submission protocol
- [x] Architecture index with tectonic map
- [x] ADR-000 (initial setup)
- [x] ADR-001 (dependency architecture — 0 cycles, coupling metrics)
- [x] `LLM Provider Abstraction Pattern` (global) — from `ModelConfig`/`ModelObject`
- [x] `LLM Fallback Cascade` (global) — from `ainvoke_with_fallback`
- [x] `Observability Data Model` (project) — `TavilyUsage`, `LLMUsage`, `ToolUsageStats`

## Tier 2: Research Tools (API Surface)
- [x] `search_and_answer` reference note
- [x] `search_and_format` reference note
- [x] `async_search_and_dedup` reference note
- [x] `crawl_and_summarize` reference note
- [x] `extract_and_summarize` reference note
- [x] `social_media_search` reference note

## Tier 3: Agents and Patterns
- [x] `HybridResearcher Agent` — fast_mode + multi_agent_mode
- [x] `Hybrid Intelligence Pattern` (global)
- [x] `Data Enrichment Flywheel` (global)
- [x] `ReAct Agent Loop` (global)
- [x] `Context Engineering Pipeline` — token, dedup, clean utilities
- [x] `Subquery Generation Pattern`

## Tier 4: Use Cases
- [x] Chatbot blueprint (Anthropic SDK + LangGraph)
- [x] Company Intelligence blueprint (ReAct + crawl)
- [x] Social Media Research blueprint
- [x] SDK vs LangGraph framework comparison

## Tier 5: Integrations and Operations
- [x] LangChain `init_chat_model` integration
- [x] LangGraph StateGraph pattern
- [x] Testing runbook (0% coverage gap analysis)
- [x] Release runbook (PyPI CI/CD)

## Index Updates (After Tier 5)
- [x] Add to `Index – Agent KB` → Projects section
- [x] Add to `Index – Agent Runtime and Capability Stack`
- [x] Create `reference/patterns/` index hub (Index – Reference Patterns)

## Evolution Log

| Date | Change | Reason | Trigger |
|---|---|---|---|
| 2026-04-18 | Initial roadmap created | First KB seeding session | Project initialization |
| 2026-04-18 | All tiers marked complete — 32 notes created across scaffold, protocols, global patterns (5), tools (6), agent, context pipeline, use cases (3), integrations, operations, ADRs, and indices | KB seeding session completion | Phase 3–5 ingestion complete |