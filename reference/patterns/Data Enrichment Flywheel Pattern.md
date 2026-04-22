---
title: Data Enrichment Flywheel Pattern
type: pattern
permalink: agent-kb/reference/patterns/data-enrichment-flywheel-pattern
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

# Data Enrichment Flywheel Pattern

## Summary
A self-reinforcing loop where web research findings are automatically indexed into the internal knowledge base after each query. Future queries hit a richer internal RAG, reducing the web search needed to fill gaps — the system gets smarter with use.

## Problem
A hybrid RAG system that treats internal and external knowledge as static inputs misses a compounding advantage: each web search contains signal that could improve all future responses.

## The Loop
```
User Query
    ↓
Hybrid Research (internal RAG + web search)
    ↓
Final Report + Web Sources
    ↓
[Index new findings back into vector store]
    ↓
Next Query → Better internal RAG → Fewer web searches needed
    ↑_______________________________↑
            (the flywheel)
```

## When It Activates
- After any `hybrid_research()` call that returns new web sources
- Filtered by confidence: only index sources with score > threshold
- Deduplicated: skip URLs already in the knowledge base

## Implementation Notes
The `tavily-agent-toolkit` documentation describes this pattern conceptually — actual KB indexing is left to the application layer (the `internal_rag_function` parameter is a callable the caller provides). The agent-toolkit provides the web research half; the application provides the KB write-back.

**Pattern boundary**: `hybrid_research()` returns `{"report": ..., "web_sources": [{"url": ..., "title": ...}], "usage": ...}`. The application takes `web_sources` and calls its own indexing pipeline.

## Value Proposition
- Reduces Tavily API credit consumption over time
- Improves response quality as institutional knowledge grows
- Creates proprietary advantage that compounds with usage

## Known Implementations
- [2026-04-18] `tavily-agent-toolkit` — described in `agents/README.MD` as design intent

## Evolution Log

| Date | Change | Reason | Trigger |
|---|---|---|---|
| 2026-04-18 | Initial pattern documented | Extracted from agent-toolkit architecture documentation | Global truth promotion |

## Relations
- derived_from [[agent-toolkit — HybridResearcher Agent]]
- related_to [[Hybrid Intelligence Pattern]]
- related_to [[Hybrid RAG Architecture – SOTA 2025-2026]]