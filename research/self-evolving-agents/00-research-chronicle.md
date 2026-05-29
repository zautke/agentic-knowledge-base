---
title: Research Chronicle - Self-Learning and Self-Evolving AI Agents
type: note
entity_type: note
permalink: research/self-evolving-agents/00-research-chronicle
created: 2026-04-21
modified: 2026-05-28
status: evergreen
tags:
- project/agent-kb
- domain/research
- domain/self-evolving-agents
- domain/agent-architecture
- status/evergreen
- source/largo
- machine/largo
aliases:
- 00-research-chronicle
---

# Research Chronicle: Self-Learning & Self-Evolving AI Agents

## Research Initiation

- [decision] Initiated systematic research into self-learning and self-evolving AI agent architectures to build an authoritative knowledge base for future agentic development
- [insight] The research was triggered by the need to understand the current landscape of autonomous agent systems that can improve themselves over time
- [fact] Research conducted January 2026, covering papers and developments through Q1 2026

## Research Scope & Questions

The investigation targeted three core questions:
1. How do AI agents learn from their own experience (self-learning)?
2. How do AI agents modify their own architecture or behavior (self-evolving)?
3. What are the practical patterns, frameworks, and pain points for production deployment?

- [decision] Chose to organize research into distinct domains (self-learning, self-evolving, memory systems, frameworks, design patterns, SOTA) for comprehensive coverage
- [insight] Separating self-learning (behavioral adaptation) from self-evolving (structural modification) proved critical -- they share mechanisms but have fundamentally different risk profiles

## Search Strategy

### Primary Sources
- ArXiv papers (2024-2026): Reflexion, SICA, EvoMAC, STELLA, Darwin Godel Machine
- Framework documentation: LangGraph, CrewAI, AutoGen/AG2, Claude Agent SDK
- Industry reports: Deloitte enterprise AI adoption surveys, MCP ecosystem metrics
- Technical blogs: Anthropic, Google DeepMind, Microsoft Research

### Search Methodology
- [decision] Used multi-pass approach -- broad survey first, then deep dives on high-signal topics
- [insight] Academic papers provided theoretical foundations, but production deployment data was scarce -- only 11% of enterprises have agents in production as of late 2025
- [decision] Prioritized production-readiness over theoretical novelty when ranking patterns

## Synthesis Approach

### Multi-Agent Documentation Architecture
- [fact] 6-role agent team deployed: Orchestrator, Research Scribe, 2 Domain Analysts, Decision Analyst, Knowledge Condenser
- [decision] Used wave-based execution (3 waves) with dependency management to ensure later notes could reference earlier findings
- [insight] The multi-agent approach itself mirrors the self-evolving agent architectures being studied -- agents specializing in different aspects of a complex task

### Knowledge Organization
- [decision] Adopted Basic Memory's observation taxonomy (fact:, decision:, insight:, requirement:) for structured knowledge capture
- [decision] Created hierarchical folder structure under research/self-evolving-agents/ with 11 notes covering all facets
- [insight] The numbered naming convention (00-10) ensures logical reading order while allowing independent access

## Sources Catalog

### Foundational Papers
- [fact] Reflexion (Shinn et al., 2023) -- Self-reflective language agents with verbal reinforcement learning
- [fact] SICA (2025) -- Self-Improving Coding Agent with modular architecture
- [fact] EvoMAC (2025) -- Multi-agent collaboration with evolutionary dynamics
- [fact] STELLA (2025) -- Self-evolving tool library for agentic systems
- [fact] Darwin Godel Machine (2025) -- Self-modifying AI systems based on Godel machine principles

### Framework & Ecosystem Sources
- [fact] LangGraph documentation and architecture guides
- [fact] CrewAI framework documentation and benchmarks
- [fact] AutoGen/AG2 Microsoft Research papers
- [fact] Claude Agent SDK documentation (Anthropic)
- [fact] Model Context Protocol (MCP) specification and adoption metrics

### Industry Data
- [fact] Deloitte State of AI in the Enterprise surveys (2024-2025)
- [fact] MCP ecosystem monthly download statistics (npm registry)
- [fact] Linux Foundation AI & Data governance announcements

## Meta-Observations

- [insight] The field is moving faster than documentation -- many cutting-edge techniques are only described in preprints or blog posts, not peer-reviewed literature
- [insight] There is a significant gap between research demonstrations (91% on benchmarks) and production readiness (11% deployment)
- [decision] Future research updates should focus on the production gap -- what specifically blocks the 89% from deploying
- [insight] The research process itself demonstrated the value of structured knowledge capture -- the observation taxonomy forced precision that free-form notes would not achieve

## Relations

- part_of [[Index – Agent KB]]

## Unresolved Relations (tracked gaps, 2026-05-28)

The chronicle above describes a planned 11-note set (00-10) under `research/self-evolving-agents/`, but only this `00-research-chronicle` note was ever created on disk. The following sibling links do not resolve. Left as tracked gaps rather than fabricated targets; if the 01-10 notes are authored later, promote these to live `chronicles`/`informs` relations.

- chronicles `[[01-self-learning-agents]]` — note never created
- chronicles `[[02-self-evolving-agents]]` — note never created
- chronicles `[[03-agentic-memory-systems]]` — note never created
- chronicles `[[04-framework-comparison]]` — note never created
- chronicles `[[05-design-patterns]]` — note never created
- chronicles `[[06-state-of-the-art-2025-2026]]` — note never created
- chronicles `[[07-critical-decisions-log]]` — note never created
- chronicles `[[08-pain-points-and-workarounds]]` — note never created
- chronicles `[[09-agent-architecture-quickref]]` — note never created
- informs `[[10-research-index]]` — note never created

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-04-21 | Initial chronicle captured (git first-commit date) | Record the research process and source catalog for the self-evolving-agents sprint | Multi-agent research sprint documentation |
| 2026-05-28 | Completed frontmatter to 7-field KB contract (added created/modified/status/entity_type, normalized tags to `domain/` taxonomy, added alias); linked orphan into root `[[Index – Agent KB]]`; demoted 10 dangling sibling links to a tracked unresolved-relations block | Note was an orphan with minimal frontmatter and 10 unresolved links to notes that were never created | KB curation hygiene sweep (operations/reference partition) |
