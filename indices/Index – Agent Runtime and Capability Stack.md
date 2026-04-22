---
title: Index – Agent Runtime and Capability Stack
type: index
permalink: agent-kb/indices/index-agent-runtime-and-capability-stack
entity_type: index
status: evergreen
tags:
- project/agent-kb
- domain/agent-systems
- index/agent-runtime
- status/evergreen
---

# Index – Agent Runtime and Capability Stack

## Purpose
Navigation hub for global notes about where runtimes, gateways, protocols, and capability frameworks fit inside agentic systems.

## Core Notes
- [[LiteLLM in Agentic Systems]]
- [[FastMCP in Agentic Systems]]
- [[MCP Capability Boundaries in Agentic Systems]]
- [[MCP Streamable HTTP and JSON-RPC Foundations]]

## Domain Tool Servers and APIs

- [[agent-toolkit — README]] — Tavily Agent Toolkit: 6 research tools (search, crawl, extract, social), HybridResearcher agent, LangChain-native
- [[agent-toolkit — search_and_answer Tool]] — Multi-query search + LLM synthesis with token guard; primary research tool
- [[agent-toolkit — async_search_and_dedup Tool]] — Parallel multi-query search with URL-based deduplication
- [[agent-toolkit — crawl_and_summarize Tool]] — Website crawler with per-page LLM summarization
- [[agent-toolkit — social_media_search Tool]] — Platform-routing social search (Reddit, X, TikTok, LinkedIn, Instagram, Facebook)

## Adjacent Reference Notes
- [[Hybrid RAG Architecture – SOTA 2025-2026]]
- [[OpenAI Codex MCP Server Interface]]

## Related Operational Indices
- [[Index – MCP Gateway Operations]]
- [[Index – MCP and Codex Operations]]

## Working Model
Use this cluster to reason about four distinct layers:
1. **Runtime / Orchestrator** — planning, memory, turn state, checkpointing, workflow control
2. **Gateway / Provider Abstraction** — model routing, failover, budgets, normalized APIs
3. **Protocol / Capability Boundary** — typed exposure of tools, prompts, resources, and remote capabilities
4. **Domain Tool Servers** — concrete implementations of browser tools, data services, operator actions, and other privileged capabilities

## Use Cases
- Decide whether a new component belongs in the runtime or at the tool boundary.
- Distinguish LiteLLM-style gateway concerns from FastMCP-style capability concerns.
- Avoid treating MCP or provider gateways as if they were full workflow engines.
- Anchor project-local agent-loop or orchestration work in reusable global references.

## Related Fundamentals
- [[Fundamental – Agent KB]]
- [[Agentic Knowledge Base System Overview]]
- [[Action Taxonomy – Index]]
- [[Document Curation Metaprompt]]

## Evolution Log
| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-04-17 | Created agent runtime and capability stack index note. | The KB had individual notes on LiteLLM, FastMCP, and MCP protocol truth, but no hub that explained how those pieces fit together in one agent-system stack. | User request to build a broader in-house knowledge cluster from current LiteLLM and FastMCP research. |
| 2026-04-17 | Linked operational MCP indices into the stack hub. | The new stack index needed explicit backlinks to the older MCP gateway and Codex operational indices so the cluster can bridge from architecture guidance into operational notes. | User request to do the index pass before reindexing the KB. |
## Sources
- Existing KB notes in `reference/tools/`, `reference/protocols/`, and `reference/rag/`
- LiteLLM official docs and release notes reviewed 2026-04-17
- FastMCP official docs and Context7 docs reviewed 2026-04-17