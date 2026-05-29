---
title: Planning - Multi-Agent Orchestration Platform
type: planning
entity_type: planning
status: active
created: 2026-04-19
modified: 2026-05-28
permalink: kb/projects/agent-toolkit/planning-multi-agent-orchestration-platform
tags:
- project/agent-toolkit
- domain/multi-agent
- domain/agent-engineering
- op/planning
- status/active
- cross-project-overlap
---

> **Cross-project overlap (flagged 2026-05-28):** This note is filed under
> `agent-toolkit` but describes the "Multi-Agent Orchestration Platform," which
> also has its own canonical project folder at `projects/multi-agent-platform/`
> (declared in that project's `project.json`). Whether these planning notes
> belong under `agent-toolkit` or `multi-agent-platform` is a cross-project
> ownership decision deferred to index reconciliation. See [[project]]
> (multi-agent-platform).

# Planning

Date: 2026-04-19
Status: Accepted for implementation
Scope: Production-grade TypeScript multi-agent orchestration platform

## Objective

Build a transparent, observable, database-backed multi-agent platform from scratch with:

- explicit orchestration using LangGraph
- role-specialized agents and quality gates
- internal and MCP-backed tool systems
- durable memory + replayable execution
- real-time dashboard for communication, traces, config, and memory observability

## Agent Topology

- `The Concierge` (Host/Coordinator): front-door orchestration and user alignment
- `The Curator` (Knowledge Curator): retrieval, synthesis, source quality
- `The Wizard` (Advanced Research): deep research and evidence collection
- `Knuth` (Code Expert): implementation and debugging
- `The Architect` (Software Architect): system decomposition and interface decisions
- `The Curmudgeon` (Quality Gate 1): adversarial reviewer and contradiction detector
- `The Overseer` (Quality Gate 2): policy/release governance and final approval

## Architecture Baseline

- Orchestration: LangGraph `StateGraph` + `Command` handoffs + interrupt/resume
- Internal system of record: PostgreSQL (events + run model)
- Semantic memory: Qdrant
- Relationship memory: Neo4j
- Hot event fan-out/cache: Redis (non-authoritative)
- Observability: OpenTelemetry spans + append-only domain event log
- Dashboard transport: WebSocket stream from event bus

## Delivery Phases

## Phase P0: Foundation and Contracts

Goal:
- Define canonical contracts before implementation.

Deliverables:
- monorepo/package structure
- TypeScript strict config + lint/test baseline
- canonical schemas:
  - `AgentConfig`
  - `ToolContract`
  - `RunEvent`
  - `MemoryRecord`
  - `ApprovalRecord`

Exit criteria:
- schema package versioned
- contract tests passing
- no implicit `any` in core packages

## Phase P1: Event-Sourced Runtime Core

Goal:
- Ship durable runtime primitives and persistence model.

Deliverables:
- PostgreSQL migrations for run/event model
- event append + query APIs
- run/session lifecycle service
- state checkpoint integration with LangGraph

Exit criteria:
- deterministic replay for a single run
- crash-resume from checkpoint
- end-to-end run metadata persisted

## Phase P2: Orchestration and Agent Runtime

Goal:
- Implement explicit handoff graph and role-specialized agents.

Deliverables:
- `Concierge` orchestration graph
- worker agent node contracts
- gate nodes (`Curmudgeon`, `Overseer`)
- interrupt/resume approval flow

Exit criteria:
- routed multi-agent run succeeds across at least 4 agents + 2 gates
- handoff events are queryable and replayable
- approval interrupt pauses and resumes safely

## Phase P3: Tooling Planes

Goal:
- Add shared internal tools and external MCP adapter layer.

Deliverables:
- internal tool SDK and registry
- per-agent tool policy (allow/deny)
- MCP adapter with timeout/retry/auth isolation
- normalized tool telemetry schema

Exit criteria:
- at least 10 shared internal tools operational
- at least 3 external MCP servers integrated through adapter
- all tool calls logged with structured inputs/outputs and latency

## Phase P4: Memory and Evaluation

Goal:
- Add memory observability and quality feedback loops.

Deliverables:
- memory read/write broker (Postgres + Qdrant + Neo4j)
- memory lineage and link tracking
- evaluation pipeline and scorecards
- contradiction/risk scoring in gate agents

Exit criteria:
- memory events visible per run and per agent
- quality gates can block progression on configurable thresholds
- evaluation artifacts persisted and queryable

## Phase P5: Dashboard and Operations

Goal:
- Provide full transparency surface and production operations.

Deliverables:
- live timeline view
- agent conversation/handoff graph
- tool execution inspector
- trace explorer
- memory panel
- config/version diff viewer
- replay UI

Exit criteria:
- operators can inspect full run state in real time
- historical replay shows equivalent step graph
- audit export contains run, tool, memory, gate, and config evidence

## Non-Goals (Initial Release)

- unrestricted swarm autonomy without routing constraints
- opaque reasoning blobs with no retention policy
- direct agent access to raw MCP servers without adapter policies

## Primary Risks and Controls

- Risk: orchestration complexity grows too quickly
  - Control: enforce explicit graph contracts and gate-based progression
- Risk: tool integrations create unstable behavior
  - Control: adapter isolation + retries + strict schema normalization
- Risk: observability overhead hurts latency
  - Control: event batching and asynchronous span exporters
- Risk: memory drift/contamination
  - Control: scoped memory writes, provenance tags, and gate checks

## Acceptance Gate for Planning-to-Build Transition

Build work starts when:

- `PLANNING.md`, `TASKS.md`, and `SESSION_HISTORY.md` are present locally
- the same artifacts are mirrored in Basic Memory
- all P0 tasks are queued with IDs and owners

## Relations

- related_to [[Tasks - Multi-Agent Orchestration Platform]]
- related_to [[Session History - Multi-Agent Orchestration Platform]]
- derived_from [[LangGraph Multi-Agent Platform Discovery - 2026-04-18]]
- overlaps [[project]] <!-- multi-agent-platform project.json; cross-project ownership to be reconciled -->

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-05-28 | Added missing frontmatter (entity_type, status, created, modified, tags); added Relations block; flagged cross-project overlap with the `multi-agent-platform` project folder. | Note had only 2 frontmatter fields and no relations; its "Multi-Agent Orchestration Platform" subject overlaps a separate canonical project. | KB curation sweep — projects/ partition. |