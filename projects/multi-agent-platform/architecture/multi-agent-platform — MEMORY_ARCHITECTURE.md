---
title: multi-agent-platform — MEMORY_ARCHITECTURE
type: reference
permalink: agent-kb/projects/multi-agent-platform/architecture/multi-agent-platform-memory-architecture
status: active
tags:
- project/multi-agent-platform
- domain/agent-engineering
- domain/multi-agent
- topic/memory
- status/discovery
---

# Multi-Agent Platform Memory Architecture

Date: 2026-04-19
Status: discovery-baseline

## Purpose
Define how memory is partitioned, who owns each layer, and how memory actions become observable events.

Hard rule:
- every memory read and write must emit a durable event

## Memory Layers

### Layer A: Working State
Purpose:
- current workflow state
- in-flight agent messages
- current decisions
- current tool outputs needed for the active run

Owner:
- orchestration runtime

Backing store:
- workflow checkpoints
- short-lived run state rows where needed

Properties:
- mutable during a run
- scoped to `run_id`
- checkpointable and resumable
- not treated as long-term truth by itself

### Layer B: Episodic Memory
Purpose:
- durable history across tasks and runs
- operator-facing run history
- approval history
- prior failures and retries
- replayable execution episodes

Owner:
- observability and control plane

Backing store:
- relational records plus canonical event ledger

Properties:
- append-oriented
- queryable by `task_id`, `run_id`, `conversation_id`, and `actor_id`
- substrate for dashboard replay and operational analytics

### Layer C: Semantic and Knowledge Memory
Purpose:
- curated knowledge
- retrieved external documents
- embeddings and semantic recall
- note-based memory for long-lived reasoning assets

Owner:
- Curator and knowledge subsystems

Backing store:
- document store
- vector index
- Basic Memory or equivalent curated knowledge surface

Properties:
- slower moving
- evidence-linked
- mutated only through explicit write paths
- writes preserve provenance and revision history

### Layer D: Configuration Memory
Purpose:
- prompts
- agent definitions
- tool manifests
- policies
- environment-scoped settings
- model routing and budget rules

Owner:
- control plane

Backing store:
- relational config tables with immutable revisions

Properties:
- versioned
- environment-aware
- never changed silently
- every effective configuration must be reconstructable for any run

## Ownership Rules
- runtime owns Layer A
- control plane owns Layers B and D
- Curator owns Layer C write discipline

## Memory Operations
Every memory action is one of:
- `read`
- `write`
- `append`
- `recall`
- `compact`
- `promote`
- `demote`
- `archive`

Every logical memory action gets a `memory_op_id`.

Matching event sequences include:
- `memory.read.requested` -> `memory.read.completed`
- `memory.write.requested` -> `memory.write.completed`

## Promotion and Demotion Rules

### Layer A -> Layer B
Promote when working state becomes a durable execution fact.
Examples:
- final task decision
- approval result
- retry reason

### Layer B -> Layer C
Promote when an episodic fact becomes reusable knowledge.
Examples:
- stable incident pattern
- validated architecture decision
- reusable troubleshooting runbook

### Layer C -> Layer D
Do not auto-promote. Configuration requires explicit operator or policy approval.

### Demotion
Demotion never deletes history. It changes active usage or retention class while preserving the event trail.

## Retrieval Rules
- working-state retrieval optimizes current run correctness
- episodic retrieval optimizes replay and near-history reasoning
- semantic retrieval optimizes relevance, citation, and synthesis
- config retrieval optimizes exact version resolution

## Memory Write Discipline
1. No silent writes.
2. No in-place destructive edits without revision history.
3. Every write records actor, reason, scope, and provenance.
4. Every cross-layer promotion records the source layer and justification.
5. Layer C writes require source references or explicit human-authored rationale.

## Observability Contract
Every memory operation must capture:
- `memory_op_id`
- `operation_type`
- `memory_layer`
- `actor_id`
- `task_id`
- `run_id`
- `scope`
- request summary
- result summary
- redaction status
- latency

These must appear in:
- canonical event ledger
- operator timeline views
- memory-specific inspection views

## Suggested Storage Shapes

### Working state
- checkpoint rows keyed by `run_id`
- optional state blobs by workflow namespace

### Episodic memory
- task summaries
- run summaries
- approval records
- transcript and message records
- denormalized task timeline cache derived from events

### Semantic memory
- documents
- chunks
- embeddings
- citation links
- provenance edges

### Config memory
- config sets
- config revisions
- rollout state
- effective configuration snapshots by environment

## Multi-Agent Access Rules
- `Concierge`: read all; write A, B, and selected D intents through control-plane APIs
- `Curator`: read all; write B and C; cannot mutate D without explicit approval path
- `Curmudgeon`: read A, B, C, and relevant D; write B as critique and review artifacts
- `Overseer`: read all; write B and D
- `Wizard`: read A, B, C, and relevant D; write A, B, and proposed C additions
- `Knuth`: read A, B, C, and relevant D; write B and proposed C additions
- `Architect`: read all; write B, C, and proposed D changes

## Basic Memory Role
Basic Memory belongs in Layer C as curated knowledge memory, not as the only memory system for the platform.

Why:
- appropriate for durable, human-readable, evidence-linked knowledge
- not sufficient by itself for run-state checkpoints, real-time task timelines, or exact configuration replay

## Initial Decisions Locked Here
- four-layer memory split
- no silent writes
- all memory operations emit events
- promotion across layers is explicit and justified
- configuration memory is versioned and immutable-by-revision

## Open Follow-Ups
- choose the document and vector store combination for Layer C
- define compaction strategy for long transcripts
- define retention policy for raw tool outputs in episodic memory
- define exact authorization rules for cross-layer writes

## Relations
- part_of [[multi-agent-platform — README]]
- related_to [[multi-agent-platform — APPROACH]]
- related_to [[multi-agent-platform — EVENT_MODEL]]

## Evolution Log
| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-04-19 | Created baseline memory architecture contract note. | The project needed clear memory ownership and observability rules before service and schema design begin. | User request to proceed with the next highest-value contract work. |