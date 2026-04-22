---
title: Hybrid Intelligence Pattern
type: pattern
permalink: agent-kb/reference/patterns/hybrid-intelligence-pattern
created: 2026-04-18
modified: 2026-04-18
entity_type: pattern
status: evergreen
tags:
- domain/agent-engineering
- domain/rag
- domain/context-engineering
- domain/patterns
- status/evergreen
---

# Hybrid Intelligence Pattern

## Summary
Combine internal knowledge (static vector store / RAG) with real-time web search into a single research pipeline. Internal knowledge provides depth and proprietary context; web search provides currency and breadth. LLM synthesizes both into a unified report.

## Problem
Internal RAG alone is stale — knowledge cuts off at index time. Web search alone lacks proprietary context. Neither alone gives an agent the full picture.

## Solution Architecture

```
User Query
    │
    ├──► Internal RAG (your data)
    │         │
    │         ▼
    │    Internal Context String
    │
    ├──► Gap Detection / Subquery Generation
    │    (LLM uses internal context to identify what's missing)
    │         │
    │         ▼
    │    Web Search Queries (targeted at gaps)
    │         │
    │         ▼
    │    Real-time Web Results
    │
    └──► Synthesis (LLM combines both contexts → report + sources)
```

## Two Modes

### Fast Mode (low latency, ~seconds)
1. Run internal RAG function
2. LLM generates subqueries aware of internal context (fills gaps, not duplicates)
3. Parallel Tavily search + deduplication
4. LLM synthesis of merged context

### Deep Mode (high quality, ~minutes)
1. Run internal RAG function
2. LLM writes a gap-filling research brief
3. Tavily Research API (multi-step AI research agent)
4. LLM synthesis of internal + deep research

## Key Design Principles
- **Gap-aware subqueries**: Web queries are generated with internal context visible — the LLM only searches for what it doesn't already know
- **Provider-agnostic**: Works with any LLM via `ModelConfig`
- **Structured output**: Both modes support Pydantic schemas for typed results
- **Usage transparency**: Full observability on both internal RAG and web search costs

## Data Enrichment Flywheel
Web findings can be fed back into the internal knowledge base after each query, improving future internal RAG results. See [[Data Enrichment Flywheel Pattern]].

## Known Implementations
- [2026-04-18] `tavily-agent-toolkit` v0.1.0 — `agents/hybrid_researcher.py` — fast_mode + multi_agent_mode
- [2026-04-18] `cookbooks/getting-started/hybrid-agent-tutorial.ipynb` — notebook walkthrough

## Evolution Log

| Date | Change | Reason | Trigger |
|---|---|---|---|
| 2026-04-18 | Initial pattern documented | Extracted from HybridResearcher agent during agent-toolkit KB seeding | Global truth promotion |

## Relations
- derived_from [[agent-toolkit — HybridResearcher Agent]]
- related_to [[Hybrid RAG Architecture – SOTA 2025-2026]]
- related_to [[LLM Provider Abstraction Pattern]]
- related_to [[Data Enrichment Flywheel Pattern]]