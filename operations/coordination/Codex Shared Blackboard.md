---
title: Codex Shared Blackboard
type: note
entity_type: note
permalink: agent-kb/operations/coordination/codex-shared-blackboard
created: 2026-04-21
modified: 2026-05-28
status: active
tags:
- project/agent-kb
- domain/codex
- domain/coordination
- artifact/blackboard
- status/active
---

# Codex Shared Blackboard

## Purpose
Live coordination surface for Foreman Codex and assistant Codex. This note is the shared progress ledger, not the source of governance truth. Governance lives in [[Codex Co-Agent Collaboration Protocol]].

## Current Participants
- Foreman Codex: user-facing owner, integrator, and verifier
- assistant Codex: delegated subtask executor through the `codex` MCP

## Current State
- Status: lease-expiry recovery drill complete
- Shared index: [[Index – Codex Co-Agent Collaboration]]
- Governing protocol: [[Codex Co-Agent Collaboration Protocol]]
- Response logging policy: every user-visible response in this collaboration must be reflected in the Blackboard or a directly related KB note.
- Most recent completed drill: [[Codex Lease Expiry Recovery Task]]

## Active Decisions
### Decisions
- Current resolved choice: use lease-based ownership rather than informal shared editing.
- Current resolved choice: keep supervisor-worker governance and use the Blackboard as a shared ledger, not a substitute for delegation.
- Current resolved choice: use task notes as the default surface for non-trivial delegated work when the Blackboard must remain stable.
- Current resolved choice: KB-only re-orientation works even when multiple task notes are active, provided the Blackboard and task notes are kept current.
- Current resolved choice: explicit `revoked` lease state is the preferred signal for unambiguous abstention drills.
- Current resolved choice: a revoked lease must be replaced by a new lease id before assistant work resumes.

### Active Assignments
- No active delegated failure-mode drill currently open.
- Most recent completed tasks: [[Codex Lease Expiry Recovery Task]], [[Codex Parallel Task A - Assistant Lease]], [[Codex Parallel Task B - Foreman Control]], [[Codex Lease Validation Task]]

### Ownership Leases
- `lease-2026-04-19-codex-expiry-001`
  owner: assistant Codex
  surface: `agent-kb/operations/coordination/tasks/codex-lease-expiry-recovery-task`
  scope: no edits authorized while lease remains revoked
  status: revoked
- `lease-2026-04-19-codex-expiry-002`
  owner: assistant Codex
  surface: `agent-kb/operations/coordination/tasks/codex-lease-expiry-recovery-task`
  scope: append to `Evidence Log` and `Verification` only
  status: released
- `lease-2026-04-19-codex-parallel-a`
  owner: assistant Codex
  surface: `agent-kb/operations/coordination/tasks/codex-parallel-task-a-assistant-lease`
  scope: append to `Evidence Log` and `Verification` only
  status: released
- `lease-2026-04-19-codex-parallel-b`
  owner: Foreman Codex
  surface: `agent-kb/operations/coordination/tasks/codex-parallel-task-b-foreman-control`
  scope: full Foreman control
  status: released
- `lease-2026-04-19-codex-task-001`
  owner: assistant Codex
  surface: `agent-kb/operations/coordination/tasks/codex-lease-validation-task`
  scope: append to `Evidence Log` and `Verification` only
  status: released

### Evidence Log
- 2026-04-18 shared Blackboard read validated from assistant Codex session.
- 2026-04-18 shared Blackboard write validated from assistant Codex session.
- 2026-04-18 cold re-orientation from index and linked notes validated from assistant Codex session.
- 2026-04-18 external prior art aligned with supervisor-worker plus structured shared state.
- 2026-04-19 live task-note lease test completed successfully through [[Codex Lease Validation Task]].
- 2026-04-19 parallel-task test completed successfully through [[Codex Parallel Task A - Assistant Lease]] and [[Codex Parallel Task B - Foreman Control]].
- 2026-04-19 KB-only re-orientation successfully reconstructed the parallel-task state from the collaboration index and linked notes.
- 2026-04-19 Foreman created [[Codex Lease Expiry Recovery Task]] to validate abstain-and-resume behavior after lease invalidation.
- 2026-04-19 Foreman revoked `lease-2026-04-19-codex-expiry-001` before assistant execution so the assistant must decide from KB state alone whether to abstain.
- 2026-04-19 assistant Codex correctly abstained under the revoked lease and identified the required Foreman action: issue a fresh valid lease.
- 2026-04-19 Foreman issued `lease-2026-04-19-codex-expiry-002` for a scoped resume-phase update.
- 2026-04-19 assistant Codex resumed under the renewed lease and Foreman verified that the resumed write stayed inside the leased task note only.


## Initial Conversation Log
### 2026-04-18
- The user requested a protocol for two Codex agents to operate as co-agents.
- Governance was clarified explicitly: the current agent is the Foreman and the `codex` MCP agent is the assistant.
- A live probe to assistant Codex established that it cannot directly verify whether it has the same MCP tool surface as the calling Codex; it can only see the tools in its own session.
- The user requested that this collaboration model be persisted in Basic Memory and extended into an evolving best-practice record.
- The user asked whether a Blackboard pattern shared memory space with an index could reduce direct context transfer.
- The user instructed that every subsequent response must be written to appropriate notes in Basic Memory to maintain a running log of actions.
- The Foreman accepted this as a standing operating rule for this collaboration thread.
- A follow-up validation step was selected: test whether assistant Codex can append to the shared Blackboard without manual context transfer.


## Working Answer On Blackboard Feasibility
Yes, with constraints.

A Blackboard in Basic Memory can reduce repeated conversational context transfer if both agents can access the same project and agree on stable note identifiers, ownership rules, and append-only status updates. It does not eliminate the need for explicit delegation, because the Foreman still has to assign scope, priorities, and decision rights.

### Validation on 2026-04-18
- assistant Codex confirmed that Basic Memory is available in its current session.
- assistant Codex successfully read `agent-kb/indices/index-codex-co-agent-collaboration`.
- assistant Codex successfully appended to `agent-kb/operations/coordination/codex-shared-blackboard` and the Foreman verified the resulting content.
- Practical status: shared read and shared write are both validated in the current session; concurrency discipline and conflict-avoidance rules still need explicit testing before parallel multi-writer use is treated as routine.


## Open Questions
- Whether assistant Codex can be instructed to update the same Basic Memory notes directly in its own session without introducing race conditions.
- Whether task-level notes should be split per assignment once concurrent work begins.
- How much of the Blackboard should be durable global truth versus ephemeral task state.

## Assistant Write Validation Log
- 2026-04-19 refinement kickoff: Foreman is tightening the Blackboard into a lease-based coordination surface with explicit sections for decisions, assignments, evidence, open questions, and ownership.

- 2026-04-18 Foreman closeout: shared Blackboard read, write, and cold re-orientation are validated; next recommended test is concurrent or sequential multi-note coordination with explicit ownership rules.

- 2026-04-18 assistant Codex cold re-orientation test succeeded from Basic Memory alone; it reconstructed governance, validation status, logging rule, open questions, and the context-transfer benefit from the collaboration index and linked notes.

- 2026-04-18 assistant Codex append test succeeded from the delegated Codex session.


## Update Rules
- Every user-visible response from the Foreman must be logged in this Blackboard or a linked task/protocol note before or during closeout.
- Foreman updates participant roster, task assignment state, decision summaries, and lease state.
- assistant Codex appends evidence or progress only to explicitly assigned sections or task notes.
- Do not replace earlier log entries; append new ones.

## Response Log
- 2026-04-19 Foreman response: logging the final verified state and closing out the lease-expiry recovery drill.
- 2026-04-19 Foreman response: promoted the completed drill into Blackboard and protocol truth after releasing the renewed lease.
- 2026-04-19 Foreman response: closed the renewed lease on the task note and normalized the task note into a done state.
- 2026-04-19 Foreman response: assistant Codex reported successful scoped action under the renewed lease and Foreman began verification.
- 2026-04-19 Foreman response: published the renewed lease and prompted assistant Codex for one narrow in-scope update.
- 2026-04-19 Foreman response: mirrored the renewed lease into the Blackboard so the resume phase is also driven from shared memory.
- 2026-04-19 Foreman response: the abstention check succeeded; recording it and issuing a fresh lease for the resume phase.
- 2026-04-19 Foreman response: synchronized the Blackboard to a single authoritative revoked-lease view before prompting assistant Codex.
- 2026-04-19 Foreman response: created [[Codex Lease Expiry Recovery Task]] and then revoked its initial assistant lease before execution to force a KB-only abstention decision.
- 2026-04-19 Foreman response: loaded the current protocol, Blackboard, and task-note template so the expiry drill updates existing structure rather than forking it.
- 2026-04-19 Foreman response: re-primed the KB rules for this session before touching the collaboration notes again.
- 2026-04-19 Foreman response: starting a lease-expiry recovery drill to validate abstain-and-resume behavior under changing lease state.
- 2026-04-19 Foreman closeout: multi-task coordination and KB-only re-orientation are validated; remaining open problem is lease expiry/renewal under longer-running concurrent tasks.
- 2026-04-19 Foreman response: assistant Codex completed its Task A lease update and is moving to a KB-only re-orientation check over the parallel-task state.
- 2026-04-19 Foreman response: updated [[Codex Parallel Task B - Foreman Control]] while keeping Task A reserved for assistant lease work.
- 2026-04-19 Foreman response: starting a multi-task coordination test with two task notes, separate ownership, and a post-task re-orientation check.
- 2026-04-19 Foreman response: removed stale duplicate assignment and lease blocks so the Blackboard contains only the current authoritative state.
- 2026-04-19 Foreman response: verified the leased task-note handoff, released the lease, and promoted the result into Blackboard and protocol truth.
- 2026-04-19 Foreman response: created [[Codex Lease Validation Task]] and activated `lease-2026-04-19-codex-task-001` for assistant Codex.
- 2026-04-19 Foreman closeout: lease model, Blackboard schema, task-note template, and collaboration index are now in place; next recommended step is a live lease-grant and task-note handoff test.
- 2026-04-19 Foreman response: removed the last duplicate governance block left behind by insert-based section edits.
- 2026-04-19 Foreman response: refining the Blackboard into a lease-based coordination model with explicit schema and task-note escalation rules.
- 2026-04-19 Foreman response: normalized the Blackboard and collaboration index so the refined schema is readable and usable.

## Related
- [[Codex Co-Agent Collaboration Protocol]]
- [[Index – Codex Co-Agent Collaboration]]
- [[Index – MCP and Codex Operations]]

## Evolution Log
| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-04-18 | Created initial shared Blackboard note and captured the first coordination session. | A durable shared progress space was needed so future Foreman/assistant Codex work can accumulate without rebuilding the entire context every time. | User request to begin logging the initial collaboration conversation and explore a Blackboard pattern. |
| 2026-04-18 | Added assistant-side Basic Memory read validation. | The Blackboard note needed to distinguish confirmed read viability from still-unverified write coordination. | Follow-up probe to verify assistant Codex access to the shared index note. |
| 2026-04-18 | Added audit-heavy response logging and a controlled write-validation target. | The Blackboard needed to reflect the user's new logging requirement and provide a safe section for assistant write testing. | User instruction to log every response plus Foreman decision to test assistant-side append behavior. |
| 2026-04-18 | Confirmed assistant-side Blackboard append behavior and normalized section structure. | The Blackboard needed explicit evidence that delegated Codex can write to the shared note and needed cleanup after incremental insert-based edits. | User request to continue with validation plus Foreman hygiene pass after observing malformed section structure. |
| 2026-04-18 | Added cold re-orientation validation from the collaboration index. | The Blackboard needed evidence that the shared notes reduce direct context transfer, not just store state. | Follow-up probe instructing assistant Codex to reconstruct collaboration state from the index and linked notes alone. |
| 2026-04-18 | Added external prior-art alignment note. | The Blackboard needed to reflect that the current architecture was checked against existing framework docs and GitHub code, not just local experiments. | User instruction to search Tavily/Context7/GitHub prior art because the pattern has existing precedent. |
| 2026-04-19 | Started lease-based refinement pass. | The shared-memory model needs stronger concurrency and ownership rules before broader use. | User request to continue refining the collaboration design. |
| 2026-04-19 | Cleaned the last duplicate governance block left by insert-based edits and logged the refined state. | The Blackboard needed one authoritative governance surface and an explicit closeout state for the refinement pass. | Foreman verification pass after seeing the remaining duplicate block in the rendered note. |
| 2026-04-19 | Completed and recorded a successful leased task-note handoff. | The Blackboard needed to reflect that the new delegation pattern works end to end and that the lease was cleanly released. | Live lease-based handoff test using [[Codex Lease Validation Task]]. |
| 2026-04-19 | Removed stale duplicate assignment and lease blocks. | The Blackboard needed to show only the current state rather than old active-lease snapshots left behind by earlier section edits. | Foreman cleanup after post-handoff verification exposed residual duplicate blocks. |
| 2026-04-19 | Completed multi-task coordination validation and logged the closeout state. | The Blackboard needed to capture that parallel task notes plus KB-only re-orientation are now validated and to identify the next untested failure mode. | Foreman closeout after successful assistant Task A update, Foreman Task B update, and KB-only re-orientation over both tasks. |
| 2026-04-19 | Started lease-expiry recovery drill. | The remaining unvalidated edge case is how the collaboration model behaves when a lease expires or is revoked mid-flow. | User instruction to continue after parallel-task validation. |
| 2026-04-19 | Added revoked-lease recovery task state to the Blackboard. | Assistant Codex needs a single authoritative shared-memory view before the abstention check is meaningful. | Foreman setup step for the lease-expiry recovery drill. |
| 2026-04-19 | Recorded successful abstention and activated the resume phase. | The drill needs to prove both stopping under invalid lease state and resuming only after explicit renewal. | assistant Codex abstention result under the revoked lease. |
| 2026-04-19 | Completed the lease-expiry recovery drill and released the renewed lease. | The Blackboard needed the final verified state showing that the assistant resumed only after renewal and that the renewed lease was closed after verification. | Foreman verification of assistant Codex's scoped renewed-lease update. |
