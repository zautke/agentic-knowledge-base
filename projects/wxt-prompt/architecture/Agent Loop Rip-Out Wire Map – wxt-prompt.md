---
title: Agent Loop Rip-Out Wire Map – wxt-prompt
type: reference
permalink: agent-kb/projects/wxt-prompt/architecture/agent-loop-rip-out-wire-map-wxt-prompt
entity_type: reference
status: draft
tags:
- project/wxt-prompt
- domain/architecture
- domain/agent-runtime
- status/draft
---

# Agent Loop Rip-Out Wire Map – wxt-prompt

## Purpose
Project-local architecture note that records the live wires connected to `useAgentLoop` so the runtime can be replaced without dropping behavior.

## Core Finding
`src/core/agent-loop/useAgentLoop.ts` is not only a chat hook. It currently concentrates UI actions, transcript state, Dexie persistence, provider dispatch, page-context injection, browser tool bridging, Ollama-specific tool behavior, auto-rename, and some provider recovery logic.

## Inbound Wires
- `entrypoints/sidepanel/App.tsx` calls `useAgentLoop(selectedModel, providers.states, activeConversationId, providers.refreshProvider, isDevToolsMode)`.
- `App.tsx` depends on the returned contract: `history`, `streaming`, `send`, `abort`, `clear`, `addMessage`, `addConversationInterruptedNotice`, `isLoaded`, and `loadedConversationId`.
- `src/hooks/useChat.ts` is a compatibility alias for `useAgentLoop` plus page-context helpers.

## Outbound Wires
- Dexie writes to `db.chatHistory` and `db.conversations`.
- Provider dispatch through `registry.get(selectedModel.providerId)` and `provider.streamChat(...)`.
- DevTools page-context fetch through `GET_PAGE_CONTEXT_HASH` and `GET_PAGE_CONTEXT` background messages.
- Browser tool execution through `EXECUTE_TOOL` background messages.
- Ollama-only tool loop through `getOllamaBrowserToolExposure()` and `runOllamaToolLoop(...)`.
- Post-turn rename through `useLLM().complete(...)`.
- Ollama cloud auth recovery through `refreshProvider('ollama')`.

## Dormant Replacement Seams
The worktree already contains a transport and kernel layer:
- `src/core/transport/types.ts`
- `src/core/transport/registryTransport.ts`
- `src/core/transport/openaiDirectTransport.ts`
- `src/kernel/contracts.ts`

These files define a cleaner evented runtime boundary, but they are not yet the live runtime.

## Replacement Guidance
- Preserve the public `useAgentLoop` surface initially unless the UI contract is intentionally changed.
- Treat LiteLLM as a provider normalization candidate, not the runtime itself.
- Treat FastMCP as a possible future browser tool boundary, not the runtime itself.
- Move one boundary at a time: timeline store, provider/runtime adapter, browser capability bridge, then post-turn cleanup.

### Relevant Global References
- [[LiteLLM in Agentic Systems]]
- [[FastMCP in Agentic Systems]]

## Evidence
- Repo wire-map doc: `/Volumes/FLOUNDER/dev/wxt-prompt/docs/architecture/agent-loop-ripout-wire-map.md`
- Hook: `/Volumes/FLOUNDER/dev/wxt-prompt/src/core/agent-loop/useAgentLoop.ts`

## Evolution Log
| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-04-17 | Created project-local wire map note for the current agent runtime. | The repo architecture doc needed a Basic Memory counterpart so replacement work can be rediscovered later. | User request to curate Basic Memory after the agent-loop research pass. |
| 2026-04-17 | Added bridge links to global LiteLLM and FastMCP references. | The local runtime note needed explicit pointers to reusable global framework references under the Bridge Protocol. | User request to create reusable global references from the current research pass. |
## Relations
- part_of [[Index – wxt-prompt Project]]
- related_to [[wxt-prompt – Project Overview]]
- related_to [[Agent Loop Replacement Planning – wxt-prompt]]
- related_to [[Readline Keybindings ADR – wxt-prompt]]
- related_to [[LiteLLM in Agentic Systems]]
- related_to [[FastMCP in Agentic Systems]]