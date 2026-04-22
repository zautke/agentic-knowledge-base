---
title: Multi-Agent Shared State and Blackboard Prior Art
type: reference
permalink: agent-kb/reference/coordination/multi-agent-shared-state-and-blackboard-prior-art
tags:
- project/agent-kb
- domain/codex
- domain/coordination
- domain/multi-agent
- domain/prior-art
- status/evergreen
---

# Multi-Agent Shared State and Blackboard Prior Art

## Purpose
Capture external prior art on multi-agent shared state, blackboard coordination, supervisor-worker orchestration, and durable memory patterns so Codex collaboration design is grounded in existing practice rather than reconstructed from scratch.

## Core Position
This has been solved many times in adjacent forms. The stable pattern is not naive shared mutable state. The stable pattern is:
- explicit coordinator or supervisor ownership for decomposition and merge decisions
- durable state or checkpoint storage for continuity
- structured shared state only where multiple agents truly need to converge on one artifact
- append-oriented logging or event history rather than silent in-place mutation

## Primary Prior Art
### LangGraph
LangGraph's current docs and code make a clear distinction between:
- graph state (`StateGraph`, shared state keys, subgraphs)
- checkpointing (`checkpointer`) for per-thread or subgraph persistence
- long-term store (`store`) for cross-thread or user-scoped memory

Operational implication:
LangGraph solves shared continuity by combining state schemas with a store/checkpointer interface, not by assuming that agents can safely mutate one ungoverned shared object.

Verified examples from docs:
- `StateGraph(MessagesState, context_schema=Context)` with a compiled `store` and `checkpointer`
- `runtime.store.search(...)` / `runtime.store.put(...)` or async equivalents for durable memory access
- parent/subgraph shared-state examples where only explicitly shared keys flow upward

### AutoGen
AutoGen's current docs emphasize saving and restoring agent or team state using `save_state()` / `load_state()`.

Operational implication:
AutoGen treats multi-agent continuity as serializable team state, which can be persisted and reloaded. This is closer to checkpoint/restore than to a freeform blackboard.

Verified examples from docs:
- `team_state = await agent_team.save_state()`
- `await agent_team.load_state(team_state)`
- team state can be serialized to JSON and reloaded later

### External pattern literature
Recent multi-agent pattern writeups converge on the same warnings:
- blackboards are useful when multiple agents must contribute to a shared artifact
- blackboards become dangerous when they are just shared mutable state without assignment, merge, or conflict rules
- production systems usually add a coordinator, assignment state, typed schemas, event logs, or all four

## Design Consequences For Codex Collaboration
### What to keep
- Foreman/assistant split for decision rights
- stable index note as re-orientation entrypoint
- Blackboard as shared progress/evidence ledger
- append-oriented evidence and evolution logs

### What to avoid
- treating the Blackboard as a replacement for delegation
- allowing multiple agents to rewrite the same summary section without ownership
- assuming read/write access implies safe concurrent editing
- relying on raw chat-context transfer when note identifiers and durable state exist

### Best-fit synthesis
The best-fit pattern for Codex is:
1. supervisor-worker governance
2. Blackboard for shared progress and evidence
3. durable protocol note for stable rules
4. per-task note or per-section ownership when parallelism increases
5. event-log mindset for history and auditability

## Specific Evidence Collected In This Session
- LangGraph docs show `store` + `checkpointer` as the memory/persistence boundary, with `StateGraph` as the state-sharing boundary.
- LangGraph code for `InMemoryStore` explicitly warns that in-memory data is lost on process exit and recommends database-backed stores like Postgres for persistence.
- AutoGen docs show teams and agents as saveable/restorable state objects rather than assuming implicit shared memory.
- External articles consistently warn that blackboards need conflict resolution, contribution tracking, or coordinator control to avoid duplicate work and race conditions.

## Sources
- LangGraph docs: `https://docs.langchain.com/oss/python/langgraph/add-memory`
- LangGraph docs: `https://docs.langchain.com/oss/python/langgraph/use-subgraphs`
- LangGraph code: `https://github.com/langchain-ai/langgraph/blob/main/libs/checkpoint/langgraph/store/memory/__init__.py`
- AutoGen docs notebook: `https://github.com/microsoft/autogen/blob/main/python/docs/src/user-guide/agentchat-user-guide/tutorial/state.ipynb`
- Supporting article: `https://www.agentpmt.com/articles/multi-agent-coordination-distributed-systems-2026`
- Supporting article: `https://contracollective.com/blog/multi-agent-orchestration-patterns`

## Related
- [[Codex Co-Agent Collaboration Protocol]]
- [[Codex Shared Blackboard]]
- [[Index – Codex Co-Agent Collaboration]]
- [[Index – MCP and Codex Operations]]
- [[MCP Capability Boundaries in Agentic Systems]]

## Evolution Log
| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-04-18 | Created prior-art reference note for multi-agent shared state and blackboard patterns. | The collaboration protocol needed grounding in external docs and code rather than only local experimentation. | User instruction to search current prior art with docs and GitHub instead of reinventing the pattern. |