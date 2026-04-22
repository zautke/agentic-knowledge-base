---
title: multi-agent-platform — PLANNING
type: reference
permalink: agent-kb/projects/multi-agent-platform/planning/multi-agent-platform-planning
status: active
tags:
- project/multi-agent-platform
- domain/agent-engineering
- domain/multi-agent
- topic/planning
- status/discovery
---

# Multi-Agent Platform Planning

Date: 2026-04-19
Status: draft

## Phase 0: discovery and architectural lock
Goals:
- finish source-backed discovery
- lock runtime, dashboard, event, memory, and protocol decisions
- produce the first stable architecture record

Exit criteria:
- approach documented locally and in KB
- initial task list created
- first session history created
- A2A compliance scope stated explicitly

## Phase 1: system contracts
Goals:
- define canonical event schema
- define trace and correlation model
- define memory architecture and lifecycle semantics
- define agent registry and tool registry contracts
- define dashboard API contracts

Exit criteria:
- event taxonomy approved
- memory tiers and ownership approved
- API sketches for tasks, traces, configs, and memory approved

## Phase 2: repo and platform bootstrap
Goals:
- create the new implementation repo in `/Volumes/MACDEV/3p/`
- scaffold Python runtime and service surface
- scaffold dashboard shell
- scaffold shared database schema and migration workflow

Exit criteria:
- repo exists
- local development boot path is documented
- runtime and dashboard can start in isolation

## Phase 3: orchestration core
Goals:
- implement agent registry
- implement workflow execution skeleton
- implement common internal toolset
- implement checkpointing, task lifecycle, and retry handling

Exit criteria:
- one end-to-end task can move through Concierge, Wizard, Curmudgeon, and Overseer
- traces and raw events are visible in durable storage

## Phase 4: memory and curation
Goals:
- implement working, episodic, semantic, and config memory layers
- implement Curator workflows
- implement memory observability surface

Exit criteria:
- memory reads and writes are logged and inspectable
- Curator can read, propose, and commit memory mutations with evidence

## Phase 5: A2A and external capability compliance
Goals:
- implement agent cards
- implement A2A task boundary
- implement MCP client and tool broker integration
- validate with inspector and samples

Exit criteria:
- at least one agent can be invoked through the A2A boundary
- compliance checks pass for the supported surface

## Phase 6: dashboard and operator controls
Goals:
- implement live communication surface
- implement trace explorer
- implement config editor
- implement memory explorer
- implement approvals queue

Exit criteria:
- operator can inspect a run live
- operator can review approvals, configs, and memory changes from the dashboard

## Phase 7: hardening
Goals:
- auth and environment isolation
- backpressure and budget controls
- failure and replay handling
- audit completeness review
- test and load validation

Exit criteria:
- production-readiness checklist exists
- failure modes are documented and test-covered where practical

## Relations
- part_of [[multi-agent-platform — README]]
- related_to [[multi-agent-platform — APPROACH]]
- related_to [[multi-agent-platform — TASKS]]
- related_to [[multi-agent-platform — SESSION_HISTORY]]

## Evolution Log
| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-04-19 | Created phased implementation planning note. | The project needed an explicit phase model before repo bootstrap and implementation begin. | User request to establish a durable paper trail for the endeavor. |