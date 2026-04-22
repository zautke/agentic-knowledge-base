---
title: Index – wxt-prompt Project
type: index
permalink: agent-kb/projects/wxt-prompt/index-wxt-prompt-project
created: 2026-04-04
modified: 2026-04-04
entity_type: index
status: evergreen
tags:
- project/wxt-prompt
- meta/index
- status/evergreen
---

# Index – wxt-prompt Project

Navigation hub for the `wxt-prompt` WXT browser extension project.
A Chrome MV3 sidepanel extension with React 19, Tailwind v4, WebSocket, and LLM chat.

## Project Root
- [[wxt-prompt – Project Overview]]
## Architecture
- [[Readline Keybindings ADR – wxt-prompt]]
- [[Agent Loop Rip-Out Wire Map – wxt-prompt]]
## Planning
- [[Agent Loop Replacement Planning – wxt-prompt]]
## Patterns
- [[Injectable Provider Tool Loop Pattern – wxt-prompt]]

## Related Domains (Global)
- [[readline-keybinding-hook-architecture]]
- [[react-controlled-textarea-cursor-restoration]]

## Entry Points
- [[Fundamental – Agent KB]]
- [[Index – Agent KB]]

## Evolution Log
| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-04-04 | Created project index | First KB entry for wxt-prompt project | Readline keybinding implementation session |
| 2026-04-17 | Added project overview, agent-loop wire map, and planning links | The project index needed current entry points for the runtime replacement work. | User request to update repo planning/session docs and curate Basic Memory. |
| 2026-04-21 | Added injectable provider tool loop pattern | Active tab context milestone completed: Anthropic + OpenAI tool loops, shared BrowserToolSchema, persistent DevTools toggle. Browser testing NOT done — unit tests only. | Milestone implementation session. |