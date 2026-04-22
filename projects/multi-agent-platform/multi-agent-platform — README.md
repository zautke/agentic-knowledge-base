---
title: multi-agent-platform — README
type: reference
permalink: agent-kb/projects/multi-agent-platform/multi-agent-platform-readme
status: active
tags:
- project/multi-agent-platform
- domain/agent-engineering
- domain/multi-agent
- status/discovery
---

# Multi-Agent Orchestration Platform — README

## What It Is
Discovery and planning context for a new production multi-agent orchestration platform. The working direction is a Python-first runtime and protocol layer, a decoupled but fundamental dashboard/control plane, and full A2A compliance.

## Current State
- Discovery baseline captured on 2026-04-19
- Architecture direction documented before implementation
- Local filesystem paper trail started in `/Volumes/MACDEV/3p/ai-interview-codex_/docs/multi-agent-platform`
- Future implementation repo planned one level up in `/Volumes/MACDEV/3p/`

## Core Decisions So Far
- Runtime and protocol plane: Python-first for v1
- Dashboard: fundamental but not tightly coupled to the runtime framework
- System of record: shared database with canonical runtime-emitted events
- Workflow engine: LangGraph inside a larger control and observability platform
- Compliance boundary: A2A is a hard requirement, MCP remains the tool and capability boundary

## Primary Notes
- [[multi-agent-platform — APPROACH]]
- [[multi-agent-platform — EVENT_MODEL]]
- [[multi-agent-platform — MEMORY_ARCHITECTURE]]
- [[multi-agent-platform — PLANNING]]
- [[multi-agent-platform — TASKS]]
- [[multi-agent-platform — SESSION_HISTORY]]

## External Research Anchors
- Mintlify `oh-my-opencode` custom agents
- LangChain and LangGraph multi-agent and persistence guidance
- LangSmith observability guidance
- A2A protocol documentation and ecosystem inspection
- GitHub reference repositories for orchestration, compliance, and observability

## Local Evidence Anchors
- `python-oop-masterclass/enterprise-agent-system/src/agents/graph.py`
- `python-oop-masterclass/enterprise-agent-system/src/api/main.py`
- `python-oop-masterclass/enterprise-agent-system/src/memory/manager.py`
- `python-oop-masterclass/enterprise-agent-system/src/infrastructure/kafka/events.py`

## Relations
- related_to [[Index – Agent Runtime and Capability Stack]]
- related_to [[MCP Capability Boundaries in Agentic Systems]]
- related_to [[FastMCP in Agentic Systems]]
- related_to [[LiteLLM in Agentic Systems]]

## Evolution Log
| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-04-19 | Created project README and discovery entrypoint. | The new platform effort needed a durable KB namespace and a stable landing page before implementation begins. | User request to establish a paper trail in files and Basic Memory. |