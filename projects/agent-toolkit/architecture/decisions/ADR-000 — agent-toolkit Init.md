---
title: ADR-000 — agent-toolkit Init
type: reference
permalink: agent-kb/projects/agent-toolkit/architecture/decisions/adr-000-agent-toolkit-init
created: 2026-04-18
modified: 2026-04-18
entity_type: reference
status: evergreen
tags:
- project/agent-toolkit
- domain/architecture
- op/decision
- status/evergreen
---

# ADR-000 — agent-toolkit Initial KB Setup

**Date**: 2026-04-18
**Status**: Accepted

## Context
The `tavily-agent-toolkit` Python package needed its patterns, agents, and architectural decisions documented in the agent-kb knowledge base for future agent discoverability and continuity.

## Decisions

### 1. Project Type: `software-lib`
The package is a production Python library (PyPI: `tavily-agent-toolkit`) with stable public API surface, CI/CD, and versioning. Standard `software-lib` scaffold applies.

### 2. jcodemunch as Primary Code Extraction Tool
All code content for KB notes is extracted via jcodemunch (`local/agent-toolkit-49aa8c6c`) rather than file reads. This yields 80-99.8% token savings and ensures symbol-level precision.

### 3. Global Truth Promotion for Core Patterns
Five patterns extracted from this package are general-purpose truths that belong in `reference/patterns/`, not this project's local folder:
- LLM Fallback Cascade
- LLM Provider Abstraction
- Hybrid Intelligence Pattern
- Data Enrichment Flywheel
- ReAct Agent Loop

### 4. Test Coverage as Known Gap
The 0% heuristic test reachability reflects that tests require real API keys and are integration-only. This is documented in the testing runbook, not treated as a defect.

### 5. Use-Cases as Examples, Not Library Code
The `use-cases/` directory contains standalone scripts demonstrating the library. They are not imported by any library code, which explains why they appear as tectonic drifters.

## Relations
- part_of [[agent-toolkit — Architecture Index]]