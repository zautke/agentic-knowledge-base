---
title: multi-agent-platform — APPROACH
type: reference
permalink: agent-kb/projects/multi-agent-platform/architecture/multi-agent-platform-approach
status: active
tags:
- project/multi-agent-platform
- domain/agent-engineering
- domain/multi-agent
- topic/architecture
- status/discovery
---

# Multi-Agent Orchestration Platform Approach

Date: 2026-04-19
Status: discovery-baseline

## Problem Statement
Build a production multi-agent orchestration platform from scratch with named specialist agents, common internal tools, external MCP integrations, specialized per-agent tooling, a live dashboard, fully transparent observability, and full A2A compliance.

The implementation repo will live one level up from the current discovery repo in `/Volumes/MACDEV/3p/`.

## Working Decisions

### Runtime language
Use Python for v1 of the runtime and protocol layer.

Why:
- the current A2A ecosystem is materially stronger in Python than in TypeScript for full protocol correctness
- the local repo already contains a Python reference system with LangGraph, checkpointing, HITL, WebSocket surface, and event taxonomy
- Python gives the shortest path to a correct A2A server/client layer while preserving room for a decoupled dashboard

### Dashboard coupling
Treat the dashboard as fundamental, but not tightly coupled to the runtime framework.

Why:
- the operator surface must observe and operate the system without inheriting workflow-framework constraints
- a shared database and event model is a better system of record than framework-native traces alone

### System of record
Use a shared database as the canonical system of record. The runtime writes canonical events. The dashboard reads and writes through service boundaries.

Why:
- the requirement is total transparency and auditability
- append-only events are the right substrate for replay, inspection, approvals, and compliance

### Orchestration engine
Use LangGraph where it is strong: workflow graphs, checkpointing, interrupts, streaming, and subgraphs. Do not treat LangGraph as the entire platform.

### A2A boundary
Build first-class A2A compliance into the runtime and deployment model, not as a later integration.

Implications:
- publish agent cards
- support A2A task lifecycle semantics
- support async execution and progress streaming
- maintain internal-to-external correlation across tasks, messages, artifacts, approvals, and tools
- validate behavior against inspector and sample implementations

## Source-Backed Rationale

### External research
- `oh-my-opencode` custom agents are a useful pattern for agent registry, permissions, and prompt composition, but not a complete production runtime
- LangGraph documentation reinforces supervisor, subagent, persistence, interrupt, and observability patterns
- LangSmith is useful for traces and evals, but should not be the sole persistence layer when the requirement is total transparency
- A2A protocol research confirms agent cards, async tasks, messages, and artifacts as the core protocol surface

### Ecosystem scan
Reference repositories reviewed during discovery:
- `langchain-ai/langgraphjs`
- `a2aproject/a2a-inspector`
- `a2aproject/a2a-samples`
- `VoltAgent/voltagent`
- `comet-ml/opik`
- `triggerdotdev/trigger.dev`

Takeaway:
- orchestration, observability, and durable control-plane concerns remain separate layers in serious systems
- compliance tooling exists, but we still need to own the system model and audit trail

### Local repo evidence
The strongest local reference is the older Python enterprise agent system under:
- `python-oop-masterclass/enterprise-agent-system/src/agents/graph.py`
- `python-oop-masterclass/enterprise-agent-system/src/api/main.py`
- `python-oop-masterclass/enterprise-agent-system/src/memory/manager.py`
- `python-oop-masterclass/enterprise-agent-system/src/infrastructure/kafka/events.py`

Key patterns already present locally:
- LangGraph `StateGraph`
- PostgreSQL checkpointing via `PostgresSaver`
- human approval and decision interrupts
- REST plus WebSocket surface
- 3-tier memory manager
- explicit event taxonomy for agent, request, approval, memory, and system events

Takeaway:
- reuse the lessons, not the codebase
- implement cleanly in a new repo

## Proposed Architecture

### Plane 1: runtime and orchestration
Owns agent execution, workflow transitions, tool invocation, retries, task state, and checkpointing.

Expected stack:
- Python
- LangGraph
- FastAPI or equivalent service surface
- background worker runtime for long-running tasks

### Plane 2: protocol and capability boundary
Owns A2A server/client support, agent cards, task lifecycle endpoints, MCP integrations, and the internal tool broker.

Expected rule:
- MCP is the tool and capability boundary
- A2A is the inter-agent and external-agent boundary

### Plane 3: control plane
Owns prompt and config versioning, agent registry, tool registry, model policy, budget policy, approval policy, and environment configuration.

### Plane 4: observability and audit
Owns the append-only event log, traces, messages, artifacts, approvals, memory operations, and replay inspection.

Hard rule:
- if it happened, it must exist as a durable event

### Plane 5: dashboard surface
Owns live agent communication, trace exploration, task inspection, config editing, memory exploration, approvals, and raw event browsing.

The dashboard is fundamental, but it is not the runtime.

## Agent Set
- `Concierge` — intake, routing, delegation, user-facing state, and task assembly
- `Curator` — retrieval, note synthesis, citation discipline, memory hygiene, and KB mutation proposals
- `Curmudgeon` — critique, completeness checks, citation checks, and revision pressure
- `Overseer` — policy, approvals, risk scoring, escalation, and compliance review
- `Wizard` — deep web research, source triangulation, freshness checks, and evidence bundles
- `Knuth` — repository analysis, code search, dependency reading, and implementation guidance
- `Architect` — decomposition, ADR drafting, interface boundaries, and tradeoff analysis

## Tooling Model

### Common internal tools
- task registry
- trace emitter
- event writer
- memory facade
- config registry
- approval service
- prompt registry
- artifact store client
- tool broker

### External MCP and service tools
- Tavily research
- GitHub search and repo inspection
- browser automation
- filesystem
- database access
- Basic Memory
- code intelligence tools such as jcodemunch

### Specialized tool ownership
- `Concierge`: routing, orchestration, user-session state, approval lookups
- `Curator`: KB search, semantic retrieval, evidence linking, note mutation
- `Curmudgeon`: rubric scoring, output diffing, citation and artifact validators
- `Overseer`: policy engine, approvals, audit queries, release gates
- `Wizard`: search, crawl, extract, source clustering, freshness checks
- `Knuth`: code search, repo graphing, dependency and blast-radius analysis
- `Architect`: topology analysis, ADR generation, interface and boundary review

## Memory Architecture

### Layer A: working state
Short-lived execution state per thread, run, and subgraph. Backed by workflow checkpoints.

### Layer B: episodic memory
Task histories, prior runs, approvals, failures, and replayable execution episodes. Backed by relational storage plus event streams.

### Layer C: semantic and knowledge memory
Curated notes, retrieved documents, embeddings, and source-backed knowledge records. Backed by a document store plus vector search.

### Layer D: configuration memory
Versioned prompts, tool manifests, policies, and agent definitions. Backed by relational storage and immutable revisions.

Required property:
- every memory read and write emits an observable event with actor, scope, payload reference, and correlation IDs

## A2A Compliance Direction
Working goals:
- expose compliant agent cards
- support external discovery and invocation
- map internal runs to A2A tasks and artifacts
- preserve async task semantics
- stream progress safely
- keep protocol-facing execution opaque where required while still preserving full internal observability in our own database

Validation plan:
- use official protocol docs as source of truth
- validate behavior against `a2a-inspector`
- compare with official samples during implementation

## Immediate Next Steps
1. Lock the event model and trace taxonomy.
2. Define the memory schema and correlation IDs.
3. Produce the new repo skeleton outside the current repo.
4. Define the first A2A-compliant service boundary.
5. Draft the dashboard API contract around traces, tasks, configs, and memory.

## Relations
- part_of [[multi-agent-platform — README]]
- related_to [[Index – Agent Runtime and Capability Stack]]
- related_to [[MCP Capability Boundaries in Agentic Systems]]
- related_to [[FastMCP in Agentic Systems]]
- related_to [[LiteLLM in Agentic Systems]]

## Sources
- Mintlify custom agents page reviewed 2026-04-19
- LangChain and LangGraph docs reviewed 2026-04-19
- LangSmith docs reviewed 2026-04-19
- A2A protocol docs reviewed 2026-04-19
- GitHub ecosystem scan reviewed 2026-04-19
- Local repo inspection and jcodemunch analysis reviewed 2026-04-19

## Evolution Log
| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-04-19 | Created initial architecture approach note for the platform effort. | The project needed a source-backed architecture record before implementation work starts. | User request to document the approach on file and in Basic Memory. |