---
title: agent-toolkit — Planning PRD Draft
type: reference
permalink: agent-kb/projects/agent-toolkit/planning/prd/agent-toolkit-planning-prd-draft
created: 2026-04-18
modified: 2026-04-18
entity_type: reference
status: draft
tags:
- project/agent-toolkit
- domain/curation
- op/planning
- status/draft
---

# Agent Toolkit KB Coverage Requirements — Draft PRD

## Goal
Ensure the agent-toolkit knowledge base contains sufficient context for any future agent to:
1. Understand what the package does and how it's structured
2. Extract any tool, agent, or pattern implementation without reading source files
3. Apply the gated curation protocol for all future submissions
4. Identify which patterns are global truths vs. project-specific
5. Understand test coverage gaps and operational runbook requirements

## Coverage Tiers

### Tier 1: Core Architecture (COMPLETE — 2026-04-18)
- [x] Package structure and topology
- [x] LLM Provider Abstraction Pattern (global)
- [x] LLM Fallback Cascade Pattern (global)
- [x] Observability Data Model
- [x] Gated KB Submission Protocol

### Tier 2: All 6 Research Tools (COMPLETE — 2026-04-18)
- [x] search_and_answer
- [x] search_and_format
- [x] async_search_and_dedup
- [x] crawl_and_summarize
- [x] extract_and_summarize
- [x] social_media_search

### Tier 3: Agents and Patterns (MOSTLY COMPLETE — 2026-04-18)
- [x] HybridResearcher Agent (fast_mode + multi_agent_mode)
- [x] Hybrid Intelligence Pattern (global)
- [x] Data Enrichment Flywheel (global)
- [x] Context Engineering Pipeline
- [ ] ReAct Agent Loop (global) — in progress
- [ ] Subquery Generation Pattern

### Tier 4: Use Cases
- [ ] Chatbot blueprint (Anthropic SDK)
- [ ] Company Intelligence blueprint
- [ ] Social Media Research blueprint
- [ ] SDK vs LangGraph comparison

### Tier 5: Operations
- [ ] Testing runbook (0% coverage gap analysis)
- [ ] Release runbook (PyPI CI/CD)

## Success Criteria
- Future agent can answer "how does search_and_answer handle token limits?" from KB without reading source
- Future agent can explain HybridResearcher fast_mode vs. multi_agent_mode tradeoffs
- Future agent knows which patterns belong in `reference/patterns/` (global) vs. project folder
- All wiki-links in project notes resolve to existing notes

## Evolution Log

| Date | Change | Reason | Trigger |
|---|---|---|---|
| 2026-04-18 | Initial PRD draft | First KB seeding session | Project initialization |