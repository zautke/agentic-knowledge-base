---
title: Tasks - Multi-Agent Orchestration Platform
type: tasks
entity_type: tasks
status: active
created: 2026-04-19
modified: 2026-05-28
permalink: kb/projects/agent-toolkit/tasks-multi-agent-orchestration-platform
tags:
- project/agent-toolkit
- domain/multi-agent
- op/tasks
- status/active
- cross-project-overlap
---

# Tasks

Date: 2026-04-19
Status legend: `[ ]` pending, `[-]` in progress, `[x]` done, `[!]` blocked

## P0 Foundation and Contracts

- [x] `P0-001` Initialize monorepo layout (`apps/`, `packages/`, `infra/`, `scripts/`)
- [x] `P0-002` Add Node + package manager constraints and lockfile
- [x] `P0-003` Configure strict TypeScript baseline and path aliases
- [x] `P0-004` Configure linting, formatting, and pre-commit checks
- [ ] `P0-005` Define `AgentConfig` schema and validation package
- [ ] `P0-006` Define `ToolContract` schema and validation package
- [ ] `P0-007` Define `RunEvent` schema and validation package
- [ ] `P0-008` Define `MemoryRecord` and `ApprovalRecord` schemas
- [ ] `P0-009` Add contract tests for all shared schemas
- [ ] `P0-010` Publish architecture decision records for core contracts

## P1 Event-Sourced Runtime Core

- [ ] `P1-001` Create PostgreSQL migration set for sessions/runs/events/spans
- [ ] `P1-002` Implement append-only event writer with idempotency key
- [ ] `P1-003` Implement run/session repository services
- [ ] `P1-004` Implement event query API (run timeline, agent slice, tool slice)
- [ ] `P1-005` Integrate LangGraph checkpoint persistence
- [ ] `P1-006` Implement replay loader from run event stream
- [ ] `P1-007` Add crash-resume integration test
- [ ] `P1-008` Add run integrity checker (missing/invalid transitions)

## P2 Orchestration and Agent Runtime

- [ ] `P2-001` Implement `Concierge` root orchestration graph
- [ ] `P2-002` Implement worker nodes for `Curator`, `Wizard`, `Knuth`, `Architect`
- [ ] `P2-003` Implement `Curmudgeon` quality gate node
- [ ] `P2-004` Implement `Overseer` governance gate node
- [ ] `P2-005` Implement explicit `Command` handoff helpers
- [ ] `P2-006` Implement interrupt/resume approval mechanism
- [ ] `P2-007` Implement per-run agent config snapshotting
- [ ] `P2-008` Add multi-agent orchestration test scenario

## P3 Tooling Planes

- [ ] `P3-001` Build internal tool SDK with typed invocation interface
- [ ] `P3-002` Build tool registry and capability discovery service
- [ ] `P3-003` Implement per-agent allowlist/denylist enforcement
- [ ] `P3-004` Implement MCP adapter abstraction
- [ ] `P3-005` Add timeout/retry policies at adapter boundary
- [ ] `P3-006` Add auth and secret isolation for MCP providers
- [ ] `P3-007` Normalize tool results and error taxonomy
- [ ] `P3-008` Emit tool-call telemetry and cost/latency metrics
- [ ] `P3-009` Integrate first external MCP server through adapter
- [ ] `P3-010` Integrate second external MCP server through adapter
- [ ] `P3-011` Integrate third external MCP server through adapter

## P4 Memory and Evaluation

- [ ] `P4-001` Implement memory broker API (read/write/list/link)
- [ ] `P4-002` Implement Qdrant semantic memory adapter
- [ ] `P4-003` Implement Neo4j relationship memory adapter
- [ ] `P4-004` Implement memory provenance and lineage fields
- [ ] `P4-005` Implement evaluation pipeline and score model
- [ ] `P4-006` Implement contradiction checks in `Curmudgeon`
- [ ] `P4-007` Implement policy checks in `Overseer`
- [ ] `P4-008` Add threshold-based blocking and escalation paths
- [ ] `P4-009` Add regression suite for memory + evaluation flows

## P5 Dashboard and Operations

- [ ] `P5-001` Build real-time run timeline API
- [ ] `P5-002` Build WebSocket event stream service
- [ ] `P5-003` Build agent communication panel
- [ ] `P5-004` Build handoff graph visualization
- [ ] `P5-005` Build trace and span explorer
- [ ] `P5-006` Build tool execution inspector
- [ ] `P5-007` Build memory observability panel
- [ ] `P5-008` Build config and agent-version diff viewer
- [ ] `P5-009` Build approvals inbox and decision panel
- [ ] `P5-010` Build replay UI for historical runs

## Cross-Cutting Quality Gates

- [ ] `Q-001` Security baseline: secret handling, least privilege, audit trails
- [ ] `Q-002` Performance baseline: p95 run latency and tool latency budgets
- [ ] `Q-003` Reliability baseline: retries, dead-letter strategy, recovery drills
- [ ] `Q-004` Observability baseline: event + trace parity checks
- [ ] `Q-005` Documentation baseline: ADRs and operator runbooks

## Current Blocking Items

- [x] `B-001` Confirm repository bootstrap choices (`pnpm` vs alternative)
- [ ] `B-002` Confirm initial deployment target (local docker vs cloud first)
- [ ] `B-003` Confirm first MCP integration set for P3 acceptance tests

## Relations

- related_to [[Planning - Multi-Agent Orchestration Platform]]
- related_to [[Session History - Multi-Agent Orchestration Platform]]

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-05-28 | Added missing frontmatter (entity_type, status, created, modified, tags); added Relations block; flagged cross-project overlap. | Note had only 2 frontmatter fields and no relations. | KB curation sweep — projects/ partition. |