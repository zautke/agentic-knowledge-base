---
title: multi-agent-platform — EVENT_MODEL
type: reference
permalink: agent-kb/projects/multi-agent-platform/architecture/multi-agent-platform-event-model
status: active
tags:
- project/multi-agent-platform
- domain/agent-engineering
- domain/multi-agent
- topic/events
- status/discovery
---

# Multi-Agent Platform Event Model

Date: 2026-04-19
Status: discovery-baseline

## Purpose
Define the canonical event ledger for the platform so runtime activity, dashboard state, approvals, and memory operations are fully observable.

Hard rule:
- if it happened, it must exist as a durable event

## Decision
V1 uses a Postgres-first event ledger with a transactional outbox pattern.

Why:
- simpler to ship correctly than introducing Kafka or another separate event bus on day one
- preserves strong transactional coupling between state changes and emitted events
- supports replay, audit, dashboard subscriptions, and later fan-out into a separate bus

## Core Principles
1. Events are append-only.
2. Events are immutable after commit.
3. Business state is reducible from event plus snapshot history.
4. Every event carries enough correlation metadata to reconstruct a run across runtime, dashboard, tools, memory, and protocol boundaries.
5. External protocol objects such as A2A tasks and artifacts map to internal event identities instead of replacing them.

## Canonical Event Envelope
Every event uses a shared envelope with these fields:
- `event_id`
- `event_version`
- `event_name`
- `event_category`
- `occurred_at`
- `recorded_at`
- `source`
- `actor`
- `trace`
- `correlation`
- `visibility`
- `payload`
- `payload_schema`

### Required identifiers
- `trace_id` — end-to-end execution path
- `span_id` — one unit of work inside the trace
- `conversation_id` — user-facing thread
- `task_id` — durable business task and dashboard join point
- `run_id` — one workflow attempt under a task
- `checkpoint_id` — workflow persistence reference
- `tool_call_id` — logical tool invocation
- `memory_op_id` — logical memory operation
- `approval_id` — approval lifecycle
- `a2a_task_id` — external A2A task mapping

## Event Categories
- `system`
- `task`
- `run`
- `agent`
- `message`
- `artifact`
- `tool`
- `memory`
- `approval`
- `policy`
- `protocol`
- `error`
- `metric`

## Minimum Event Taxonomy

### Task
- `agent.task.created`
- `agent.task.accepted`
- `agent.task.rejected`
- `agent.task.completed`
- `agent.task.failed`
- `agent.task.canceled`

### Run
- `agent.run.started`
- `agent.run.checkpointed`
- `agent.run.resumed`
- `agent.run.completed`
- `agent.run.failed`
- `agent.run.retried`

### Agent
- `agent.node.entered`
- `agent.node.exited`
- `agent.delegation.created`
- `agent.delegation.accepted`
- `agent.delegation.rejected`
- `agent.decision.recorded`

### Message
- `agent.message.created`
- `agent.message.redacted`
- `agent.message.published`

### Artifact
- `agent.artifact.created`
- `agent.artifact.updated`
- `agent.artifact.published`

### Tool
- `tool.call.requested`
- `tool.call.started`
- `tool.call.completed`
- `tool.call.failed`
- `tool.call.redacted`

### Memory
- `memory.read.requested`
- `memory.read.completed`
- `memory.write.requested`
- `memory.write.completed`
- `memory.compaction.completed`
- `memory.recall.materialized`

### Approval
- `approval.requested`
- `approval.assigned`
- `approval.granted`
- `approval.rejected`
- `approval.expired`

### Policy
- `policy.check.started`
- `policy.check.passed`
- `policy.check.failed`
- `policy.override.recorded`

### Protocol
- `a2a.card.published`
- `a2a.task.received`
- `a2a.task.status.updated`
- `a2a.artifact.mapped`
- `mcp.session.opened`
- `mcp.tool.bound`

### Error and metric
- `system.error.recorded`
- `metric.latency.recorded`
- `metric.token_usage.recorded`
- `metric.cost.recorded`

## Storage Model

### Canonical ledger: `events`
Suggested columns:
- identifiers and timestamps
- trace and correlation fields
- source and actor fields
- `payload_schema`
- `payload_json`
- `visibility_json`

Recommended indexes:
- `(task_id, occurred_at)`
- `(run_id, occurred_at)`
- `(trace_id, occurred_at)`
- `(event_name, occurred_at desc)`
- `(approval_id, occurred_at)`
- `(memory_op_id, occurred_at)`
- `(tool_call_id, occurred_at)`
- `(a2a_task_id, occurred_at)`

### Transactional handoff: `event_outbox`
Tracks downstream delivery state for subscribers and stream publication.

### Optional reductions: `event_snapshots`
Supports latest task, run, approval, and memory summaries for faster dashboard reads.

## Redaction Model
Every event supports controlled visibility.

Redaction states:
- `none`
- `pending`
- `partial`
- `complete`

Rule:
- retain the event and redact payload fields; never silently drop the event

## A2A Mapping Rules
- internal `task_id` remains canonical
- external `a2a_task_id` is an attached protocol mapping
- A2A artifacts map to internal artifacts plus `a2a_artifact_id`
- protocol updates are part of the event chain, not the only status record

## Dashboard Query Shapes
The event model must make these cheap:
- task timeline by `task_id`
- current run trace by `run_id`
- agent communication stream by `conversation_id`
- tool audit by `tool_call_id`
- approval history by `approval_id`
- memory activity by `memory_op_id`
- external protocol correlation by `a2a_task_id`

## Initial Decisions Locked Here
- Postgres-first canonical ledger
- transactional outbox
- append-only immutable events
- internal identifiers remain canonical over external protocol IDs
- redaction modifies payload visibility, not event existence

## Open Follow-Ups
- choose exact ID format: ULID vs UUIDv7
- choose snapshot refresh strategy
- choose retention policy for raw model text and large artifacts
- define the artifact store contract for oversize payloads

## Relations
- part_of [[multi-agent-platform — README]]
- related_to [[multi-agent-platform — APPROACH]]
- related_to [[multi-agent-platform — MEMORY_ARCHITECTURE]]

## Evolution Log
| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-04-19 | Created canonical event ledger contract note. | The project needed a concrete event model before database and API design can start. | User request to proceed with the next highest-value contract work. |