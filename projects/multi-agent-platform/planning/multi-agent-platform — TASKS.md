---
title: multi-agent-platform — TASKS
type: reference
permalink: agent-kb/projects/multi-agent-platform/planning/multi-agent-platform-tasks
status: active
tags:
- project/multi-agent-platform
- domain/agent-engineering
- domain/multi-agent
- topic/tasks
- status/discovery
---

# Multi-Agent Platform Tasks

Date: 2026-04-19
Status: active discovery backlog

## Done
- [x] Research custom-agent patterns from `oh-my-opencode`
- [x] Research LangGraph multi-agent and production workflow guidance
- [x] Research A2A protocol direction and compliance requirements
- [x] Scan current ecosystem references through GitHub search
- [x] Inspect this repo with jcodemunch and direct file reads
- [x] Decide on Python-first runtime direction for v1
- [x] Decide that the dashboard is fundamental but decoupled
- [x] Start the local paper trail

## In Progress
- [x] Define canonical event schema and trace taxonomy
- [x] Define memory architecture in enough detail to support implementation
- [ ] Define the common internal toolset contract
- [ ] Define external MCP tool inventory and ownership model
- [ ] Define per-agent specialized tool access
- [ ] Define the first A2A compliance slice for implementation
- [x] Mirror the project record into Basic Memory

## Next
- [ ] Choose the new repo name and create it in `/Volumes/MACDEV/3p/`
- [ ] Draft the runtime package layout
- [ ] Draft the dashboard package layout
- [ ] Draft the database schema for tasks, traces, events, configs, approvals, and memory
- [ ] Draft the first operator workflow from intake to approval

## Open Questions
- [ ] Which frontend stack should own the dashboard shell
- [ ] Whether to use Postgres-only first or Postgres plus separate event transport from day one
- [ ] Which artifact store to use for large payloads and replay assets
- [ ] Whether all agents are exposed through A2A or only selected boundary agents in v1

## Relations
- part_of [[multi-agent-platform — README]]
- related_to [[multi-agent-platform — APPROACH]]
- related_to [[multi-agent-platform — PLANNING]]

## Evolution Log
| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-04-19 | Created initial task ledger for discovery and bootstrap work. | The project needed an explicit backlog to convert research into implementation contracts. | User request to establish a durable paper trail for the endeavor. |