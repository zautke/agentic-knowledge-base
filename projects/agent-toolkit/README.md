---
title: README
type: reference
permalink: agent-kb/projects/agent-toolkit/readme
created: 2026-04-18
modified: 2026-04-18
entity_type: reference
status: evergreen
tags:
- project/agent-toolkit
- domain/agent-engineering
- domain/context-engineering
- status/evergreen
---

# Tavily Agent Toolkit — KB Overview

## What It Is
Production Python package (`tavily-agent-toolkit` v0.1.0, published to PyPI) providing research tools, agents, and context engineering utilities for building AI applications with real-time web data. Maintained by the Tavily FDE team inside the `tavily-cookbook` monorepo.

## Repository
- **Path**: `/Volumes/MACDEV/3p/tavily-cookbook/agent-toolkit`
- **jcodemunch index**: `local/agent-toolkit-49aa8c6c` (190 symbols, 30 files)
- **Python support**: 3.10, 3.11, 3.12

## Package Surface

| Layer | Components |
|---|---|
| **Tools** | `search_and_answer`, `search_and_format`, `search_dedup`, `crawl_and_summarize`, `extract_and_summarize`, `social_media_search` |
| **Agents** | `HybridResearcher` (fast mode + multi-agent mode) |
| **Use Cases** | Chatbot, Company Intelligence, Social Media Research — each in Anthropic SDK + LangGraph |
| **Utilities** | Retry, fallback, token counting, dedup, content cleaning, subquery generation, synthesis |
| **Models** | `ModelConfig`, `ModelObject`, `OutputSchema`, `TavilyUsage`, `LLMUsage`, `ToolUsageStats` |

## Core Design Principles
1. **Bring Your Own Model** — all tools accept `ModelConfig` with 20+ providers via LangChain
2. **Context as First Class** — token management, dedup, and content cleaning built in
3. **Observability Native** — usage tracking (Tavily credits, LLM tokens, response times) in every tool
4. **Hybrid Intelligence** — internal RAG + real-time web as the primary pattern
5. **Multi-Framework** — identical use cases in Anthropic SDK and LangGraph side-by-side

## KB Structure
- [[agent-toolkit — Architecture Index]] — module topology, tectonic map, ADRs
- [[agent-toolkit — Gated KB Submission Protocol]] — gate system for all future ingestion
- [[agent-toolkit — Planning Roadmap]] — ingestion tiers and status
- `knowledge/tools/` — per-tool reference notes
- `knowledge/agents/` — agent architecture notes
- `knowledge/patterns/` — project-specific pattern implementations
- `knowledge/use-cases/` — production blueprint notes
- `knowledge/integrations/` — framework integration notes
- `operations/runbooks/` — testing, release, ops runbooks

## Relations
- part_of [[Index – Agent KB]]
- related_to [[Index – Agent Runtime and Capability Stack]]
- related_to [[Hybrid RAG Architecture – SOTA 2025-2026]]