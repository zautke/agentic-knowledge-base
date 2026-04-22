---
title: wxt-prompt – Project Overview
type: reference
permalink: agent-kb/projects/wxt-prompt/wxt-prompt-project-overview
entity_type: reference
status: draft
tags:
- project/wxt-prompt
- domain/browser-extension
- domain/agent-runtime
- status/draft
---

# wxt-prompt – Project Overview

## Purpose
Project-local overview for the `wxt-prompt` browser extension codebase.

## Current System Shape
- Chrome MV3 sidepanel extension built with WXT, React, and TypeScript.
- Multi-provider chat runtime with provider registry entries for Ollama, OpenAI, Anthropic, Gemini, and Codex.
- Conversation state persists in Dexie via `chatHistory`, `conversations`, `providerSettings`, and `appState`.
- Background runtime exposes browser tools and page-context retrieval through `browser.runtime.sendMessage(...)` handlers.
- The canonical chat runtime boundary is currently `src/core/agent-loop/useAgentLoop.ts`.

## Current Focus
The active architecture focus is replacing the current agent loop safely.

The main repo-level artifacts created during this pass are:
- `docs/architecture/agent-loop-ripout-wire-map.md`
- `TASKS.md`
- `PLANNING.md`
- `SESSION_HISOTRY.md`

## Important Local Boundaries
- `entrypoints/sidepanel/App.tsx` is the main UI composition root.
- `src/core/agent-loop/useAgentLoop.ts` is the active runtime seam.
- `src/core/transport/` is present but not yet wired into the live runtime.
- `entrypoints/background.ts` owns the current browser tool and page-context message handlers.

## Related Project Notes
- [[Index – wxt-prompt Project]]
- [[Readline Keybindings ADR – wxt-prompt]]
- [[Agent Loop Rip-Out Wire Map – wxt-prompt]]
- [[Agent Loop Replacement Planning – wxt-prompt]]

## Evidence
- Repo root: `/Volumes/FLOUNDER/dev/wxt-prompt`
- Wire map note: `/Volumes/FLOUNDER/dev/wxt-prompt/docs/architecture/agent-loop-ripout-wire-map.md`

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-04-17 | Created project overview note and linked current runtime/planning artifacts. | The project index had a stub root note and no current overview for the agent-loop replacement work. | User request to update repo tracking docs and curate Basic Memory. |

## Relations
- part_of [[Index – wxt-prompt Project]]
- related_to [[Readline Keybindings ADR – wxt-prompt]]
- related_to [[Agent Loop Rip-Out Wire Map – wxt-prompt]]
- related_to [[Agent Loop Replacement Planning – wxt-prompt]]