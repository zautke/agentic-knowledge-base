---
title: Agent Loop Replacement Planning – wxt-prompt
type: reference
permalink: agent-kb/projects/wxt-prompt/planning/agent-loop-replacement-planning-wxt-prompt
entity_type: reference
status: draft
tags:
- project/wxt-prompt
- domain/planning
- domain/agent-runtime
- status/draft
---

# Agent Loop Replacement Planning – wxt-prompt

## Goal
Replace the current agent loop with a runtime that is easier to reason about, easier to test, and easier to reconnect to the extension's UI, provider, and browser-capability boundaries.

## Decisions From This Research Pass
- LangGraph is the strongest reference model for the replacement architecture: runtime-owned state, explicit tool steps, durable execution, and evented orchestration.
- LiteLLM looks useful as a provider shim or routing layer, not as the runtime that replaces the agent loop.
- FastMCP looks useful if browser tools and page-context capabilities move behind a typed MCP boundary.
- The first migration should preserve the `useAgentLoop` public surface unless the sidepanel contract is intentionally revised.

## Planned Sequence
### 1. Preserve behavior while isolating seams
- Define the exact runtime events the replacement loop must emit.
- Define how those events map back into `DbConversationItem` mutations.
- Extract page-context and browser-tool policy out of the hook.
- Extract post-turn cleanup out of the hook.

### 2. Swap orchestration internals
- Replace direct provider-loop logic inside the hook with a dedicated runtime engine.
- Reuse or revise the dormant transport abstraction only after the live event and persistence contract is clear.

### 3. Normalize provider and tool handling
- Remove the current Ollama-only tool-loop asymmetry.
- Decide whether LiteLLM belongs at the provider boundary, transport boundary, or a later phase.
- Decide whether browser tools stay on background messages or move behind FastMCP/MCP.

## Open Questions
- Should the first replacement runtime reduce directly into Dexie/timeline rows, or into a canonical runtime state first?
- Should `addMessage()` remain a UI escape hatch, or become a runtime event?
- Should auto-rename remain part of the runtime, or move into a separate post-turn service?

## Related Project Notes
- [[Index – wxt-prompt Project]]
- [[wxt-prompt – Project Overview]]
- [[Agent Loop Rip-Out Wire Map – wxt-prompt]]

## Global References
- [[LiteLLM in Agentic Systems]]
- [[FastMCP in Agentic Systems]]

## Evidence
- Repo planning doc: `/Volumes/FLOUNDER/dev/wxt-prompt/PLANNING.md`
- Repo wire-map doc: `/Volumes/FLOUNDER/dev/wxt-prompt/docs/architecture/agent-loop-ripout-wire-map.md`

## Evolution Log
| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-04-17 | Created project planning note for agent-loop replacement work. | The project index had no planning notes and the current runtime research needed a durable project-local plan entry. | User request to update repo planning docs and curate Basic Memory. |
| 2026-04-17 | Added bridge links to global LiteLLM and FastMCP references. | The local planning note needed explicit links to the new global tool/framework references under the Bridge Protocol. | User request to create reusable global references from the current research pass. |
## Relations
- part_of [[Index – wxt-prompt Project]]
- related_to [[wxt-prompt – Project Overview]]
- related_to [[Agent Loop Rip-Out Wire Map – wxt-prompt]]
- related_to [[LiteLLM in Agentic Systems]]
- related_to [[FastMCP in Agentic Systems]]