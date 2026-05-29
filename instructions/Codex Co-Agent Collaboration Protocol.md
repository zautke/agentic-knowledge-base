---
title: Codex Co-Agent Collaboration Protocol
type: instruction
permalink: agent-kb/instructions/codex-co-agent-collaboration-protocol
tags:
- project/agent-kb
- domain/codex
- domain/coordination
- domain/multi-agent
- status/evergreen
- source/largo
- machine/largo
---

# Codex Co-Agent Collaboration Protocol

## Purpose
Define the working protocol for a `Foreman Codex -> assistant Codex` operating model so multi-agent Codex work remains bounded, auditable, and low-overhead.

## Agent Curation Metaprompt
ROLE: You are a curator for the Codex co-agent collaboration protocol. This note is a living instruction for how one agent should delegate to, supervise, and consume work from another Codex agent.

GOVERNING PRINCIPLES:
1. ACCUMULATE, NEVER SUMMARIZE AWAY. Preserve concrete coordination failures, fixes, and examples.
2. STRUCTURED INCREMENTAL UPDATES ONLY. Add new operational truths to named sections.
3. EVIDENCE-LINKED ENTRIES. Record date, context, example, and source for each new truth.
4. SELF-REFERENTIAL VERIFICATION. Check for contradiction with existing Codex or MCP notes before editing.
5. EVOLUTION LOG IS MANDATORY. Append every material update.
6. TRIGGER CONDITIONS FOR UPDATE. Update this note when: a new coordination failure occurs, a new handoff format is validated, a Blackboard workflow is tested, tool-surface assumptions change, or the user requests a revision.
7. WHAT NOT TO CHANGE. Do not remove prior operational truths; deprecate them explicitly instead.
8. QUALITY SIGNAL TRACKING. Record which parts of the protocol reduced duplication, ambiguity, or failed handoffs.

## Governance Model
- The `Foreman` owns user communication, final decisions, task decomposition, integration, and verification.
- The `assistant` owns only the delegated slice and does not broaden scope silently.
- The assistant never talks to the user directly.
- The Foreman remains accountable for correctness even when the assistant executes work.

## Role Boundaries
### Foreman responsibilities
- Frame the problem and choose delegation boundaries.
- Package only decision-relevant context.
- Assign exact ownership of files, analysis scope, or verification scope.
- Integrate returned work and resolve cross-cutting tradeoffs.
- Perform or supervise final verification before reporting completion.

### Assistant responsibilities
- Execute the bounded subtask.
- Stay inside the declared write or read scope.
- Return evidence, not just conclusions.
- Escalate ambiguity rather than freelancing into adjacent systems.

## Foreman -> Assistant Handoff Format
```text
TASK
OBJECTIVE
WHY NOW
CONTEXT
WRITE SCOPE
DO NOT TOUCH
DELIVERABLE
VERIFICATION
OPEN QUESTIONS
STOP CONDITION
```

## Assistant -> Foreman Return Format
```text
STATUS
RESULT
CHANGES
EVIDENCE
RISKS
OPEN QUESTIONS
RECOMMENDED NEXT STEP
```

## Blackboard Pattern Decision
### Current position
A Blackboard-style shared memory is viable if both agents can access the same Basic Memory project and if the Blackboard is treated as a durable progress ledger rather than a magic substitute for delegation.

### What the Blackboard is good for
- Shared task status
- Decision log
- Evidence log
- Open questions queue
- Canonical links to active notes and related protocols

### What the Blackboard does not replace
- Initial task assignment
- Decision rights
- Explicit ownership of write scope
- Fresh probing of tool capability differences between sessions

## Recommended Blackboard Rules
- User-visible responses should be logged to the Blackboard or a linked task/protocol note as part of normal operation when the collaboration is being run in audit-heavy mode.
- The Foreman owns the Blackboard index and assignment state.
- Each delegated task should have one owning note or one clearly assigned section to avoid concurrent edit collisions.
- The assistant should append evidence and status, not rewrite global summaries.
- Any claim about Codex tool capability parity must be explicitly probed in-session; parity is never assumed.
- Agents should pass note identifiers and expected sections, not whole conversational history, whenever possible.
- Prefer lease-based ownership over informal assumptions about who is editing what.
- Split from the shared Blackboard into task notes early when one task begins to dominate the ledger.

## Lease Model
### Core idea
Shared access is allowed, but active ownership is leased rather than assumed. A lease is the right to modify one declared Blackboard surface for a bounded period.

### Lease fields
- `lease_id`
- `owner`
- `surface` — note path or named section
- `scope` — what edits are allowed
- `acquired_at`
- `expires_at`
- `status` — `active`, `released`, `expired`, `revoked`
- `heartbeat` — optional freshness timestamp for long tasks

### Lease rules
- The Foreman grants, extends, and revokes leases.
- Only one active writer lease should exist for a mutable summary surface at a time.
- Append-only logs may allow multiple writers only if each entry is independent and timestamped.
- If a lease expires, the Foreman reclaims the surface until a new lease is granted.
- The assistant should not edit outside its lease even if it can technically access the note.

## Blackboard Schema
### Foreman-owned sections
- `Decisions`
- `Active Assignments`
- `Ownership Leases`
- `Current State`

### Shared append-only sections
- `Evidence Log`
- `Response Log`
- `Validation Log`
- `Open Questions` (append new questions rather than rewriting prior ones)

### Notes on ownership
- The assistant should generally append facts, evidence, and status updates.
- The Foreman should collapse or rewrite operational state when needed because integration authority lives there.

## Escalation To Task Notes
Create a dedicated task note when any of the following becomes true:
- more than one active assignment exists at once
- a task needs more than one substantial assistant update
- the same artifact would otherwise be edited by both agents
- the evidence log for one task starts to drown out global Blackboard state
- the task requires its own acceptance criteria or verification plan

Recommended task-note location:
- `operations/coordination/tasks/`

Task notes should inherit the same pattern:
- Foreman-owned state fields
- append-only evidence log
- explicit lease or ownership section
- backlink to [[Codex Shared Blackboard]]


## Operational Truths Verified In This Session
- [2026-04-18] A live probe through the `codex` MCP returned: the assistant Codex cannot directly verify whether it has the same MCP tools as the calling Codex and can only reason from the tools available in its own session.
- [2026-04-18] The most reliable authority split is `Foreman for global control, assistant for bounded local execution`, not a peer model.
- [2026-04-18] Basic Memory can serve as a shared Blackboard only if the collaboration protocol defines ownership, append discipline, and an index entrypoint.
- [2026-04-18] assistant Codex was able to read `agent-kb/indices/index-codex-co-agent-collaboration` from its own session, which confirms that the shared Blackboard design is operationally viable at least for direct KB reads.
- [2026-04-18] assistant Codex successfully appended to `agent-kb/operations/coordination/codex-shared-blackboard` and the Foreman verified the resulting note content, which confirms that shared Blackboard writes are operationally viable in the current session.
- [2026-04-18] assistant Codex successfully re-oriented itself from the collaboration index and linked notes without needing the Foreman to replay the chat, which confirms that the Blackboard+index pattern materially reduces direct context transfer for ongoing work.
- [2026-04-19] assistant Codex successfully executed a leased task-note update against `agent-kb/operations/coordination/tasks/codex-lease-validation-task` while staying inside the declared scope, and the Foreman verified the resulting note content.
- [2026-04-19] Task-scoped notes plus released leases provide a cleaner delegation pattern than direct edits to the shared Blackboard for non-trivial work.
- [2026-04-19] Two simultaneous task notes with separate ownership remained coherent under KB-only re-orientation, which confirms that the collaboration index plus Blackboard can scale beyond a single delegated task.
- [2026-04-19] Foreman-owned and assistant-leased task notes can coexist without ownership confusion when the Blackboard explicitly summarizes assignments and lease state.
- [2026-04-19] assistant Codex correctly abstained from editing when the only relevant lease in the task note and Blackboard was `revoked`, which validates lease-state enforcement through KB-only re-orientation.
- [2026-04-19] assistant Codex resumed work only after Foreman issued a fresh lease id and recorded it in both the task note and Blackboard, which validates explicit renewal as the handoff boundary after revocation.

## Action Taxonomy Mapping
- `Create Node`: create protocol, Blackboard, and related operational notes.
- `Update Node`: append new coordination truths, failures, and tested handoff formats.
- `Relate Nodes`: link Codex coordination notes to MCP, runtime, and memory-system references.
- `Create Index-Hubs`: maintain a stable hub for Codex collaboration artifacts.
- `Split / Merge`: split tactical blackboard state from durable best-practice once either becomes too large.
- `Refile / Rename`: move local experiments into global protocol notes once generalized.
- `Deprecate / Archive`: mark obsolete handoff formats or coordination assumptions explicitly.
- `Hygiene / Compression`: normalize tags, links, and unresolved references without deleting evidence.

## Quality Signals
- Reduced duplicate work between Foreman and assistant
- Lower handoff ambiguity
- Faster recovery from stalled assistant work
- Fewer scope violations
- Shorter context payloads without loss of correctness

## Related
- [[Multi-Agent Shared State and Blackboard Prior Art]]
- [[Codex Coordination Task Note Template]]
- [[Index – Codex Co-Agent Collaboration]]
- [[Codex Shared Blackboard]]
- [[Index – MCP and Codex Operations]]
- [[OpenAI Codex MCP Server Interface]]
- [[MCP Capability Boundaries in Agentic Systems]]
- [[Agentic Knowledge Base System Overview]]
- [[Document Curation Metaprompt]]

## Evolution Log
| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-04-18 | Created the initial Codex co-agent collaboration protocol. | A durable protocol was needed for Foreman/assistant Codex coordination and for grounding future Blackboard workflows in the KB. | User request to formalize co-agent best practices and persist them in Basic Memory. |
| 2026-04-18 | Added live Basic Memory read validation from assistant Codex. | The Blackboard design needed a direct operational check, not just an architectural guess. | Follow-up probe to verify that assistant Codex could read the shared collaboration index in `agent-kb`. |
| 2026-04-18 | Added audit-heavy response logging rule. | The collaboration protocol needed to capture the user's requirement that every response be logged into Basic Memory during this thread. | User instruction to keep a running log of all subsequent actions and responses. |
| 2026-04-18 | Added assistant-side Blackboard write validation. | The protocol needed to distinguish theoretical Blackboard design from confirmed cross-agent write capability. | Follow-up probe asking assistant Codex to append to the shared Blackboard and report the result. |
| 2026-04-18 | Added cold re-orientation validation from the collaboration index. | The protocol needed evidence that the Blackboard and index reduce context-transfer overhead, not just that they support storage. | Follow-up probe instructing assistant Codex to reconstruct current state using only Basic Memory notes. |
| 2026-04-19 | Added lease model, Blackboard schema, and task-note escalation guidance. | The protocol needed stronger concurrency discipline and an explicit escape hatch from the shared Blackboard to task-scoped notes. | User request to continue refining the collaboration design. |
| 2026-04-19 | Validated lease-based task-note handoff end to end. | The protocol needed proof that a leased task note is a better delegation surface than direct Blackboard editing for non-trivial work. | Live lease-grant, assistant update, and Foreman verification against `Codex Lease Validation Task`. |
| 2026-04-19 | Validated multi-task coordination with KB-only re-orientation. | The protocol needed evidence that the same design still works when two task notes are active under separate ownership. | Live parallel-task test using `Codex Parallel Task A - Assistant Lease` and `Codex Parallel Task B - Foreman Control`, followed by KB-only re-orientation. |
| 2026-04-19 | Validated revoked-lease abstention and renewed-lease resume behavior. | The protocol needed proof that lease state is not just descriptive but actually gates assistant action through shared memory. | Live failure-mode drill using [[Codex Lease Expiry Recovery Task]], revoked lease `lease-2026-04-19-codex-expiry-001`, and renewed lease `lease-2026-04-19-codex-expiry-002`. |

## Sources
- Live user conversation on 2026-04-18 about Foreman Codex and assistant Codex coordination
- Live `codex` MCP probe on 2026-04-18 about tool-surface visibility
- Existing KB notes: `Index – MCP and Codex Operations`, `OpenAI Codex MCP Server Interface`, `MCP Capability Boundaries in Agentic Systems`