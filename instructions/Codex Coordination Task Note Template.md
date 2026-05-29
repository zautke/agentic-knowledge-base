---
title: Codex Coordination Task Note Template
type: instruction
permalink: agent-kb/instructions/codex-coordination-task-note-template
tags:
- project/agent-kb
- domain/codex
- domain/coordination
- domain/multi-agent
- status/evergreen
- source/largo
- machine/largo
---

# Codex Coordination Task Note Template

## Purpose
Template and usage guidance for per-task coordination notes used when the shared Blackboard is no longer the right granularity.

## When To Use
Create a task note when:
- more than one active assignment exists
- a task requires multiple updates from assistant Codex
- a task needs its own verification plan
- a shared note would otherwise create write contention

## Recommended Location
- `operations/coordination/tasks/`

## Recommended Structure
```markdown
# <Task Title>

## Purpose

## Status
- state: proposed | active | blocked | done
- owner: Foreman | assistant Codex
- priority:
- linked blackboard: [[Codex Shared Blackboard]]

## Objective

## Scope
- in:
- out:

## Ownership Lease
- lease_id:
- owner:
- surface:
- acquired_at:
- expires_at:
- status:

## Acceptance Criteria
- ...

## Evidence Log
- timestamped append-only entries

## Open Questions
- append-only unless Foreman resolves them into Decisions

## Decisions
- Foreman-owned summary of resolved choices

## Verification
- commands run
- outcomes

## Evolution Log
| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
```

## Rules
- Evidence is append-only.
- Decisions are Foreman-owned.
- The assistant updates only the leased task note or explicitly assigned sections.
- Task notes should link back to the Blackboard and the protocol note.

## Related
- [[Codex Co-Agent Collaboration Protocol]]
- [[Codex Shared Blackboard]]
- [[Index – Codex Co-Agent Collaboration]]

## Evolution Log
| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-04-19 | Created coordination task-note template. | The shared Blackboard needed a clear escape hatch for task-level state once concurrency or note volume increases. | Ongoing refinement of the Codex collaboration protocol. |
