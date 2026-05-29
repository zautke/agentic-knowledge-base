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
- [[Full Healing Plan – wxt-prompt (2026-04-22)]]

- [[Agent Loop Replacement Planning – wxt-prompt]]
## Patterns
- [[Injectable Provider Tool Loop Pattern – wxt-prompt]]

## Browser Testing
- [[WXT Extension E2E Testing Runbook — wxt-prompt]]
- [[Ollama Provider Connection Timing Reference — wxt-prompt]]
- [[Chrome Extension Playwright Locator Patterns — wxt-prompt]]
- [[Live Test Failure Map – wxt-prompt (2026-04-26)]]
- [[wxt-prompt Live Test Catastrophic Failure Retrospective (2026-04-26)]]

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
| 2026-04-21 | Added browser testing section: E2E runbook, Ollama timing reference, Playwright locator patterns | Browser verification completed with milestone.live.spec.ts. Scenarios 1-3 PASS. t+2 KB documentation requested. | User post-milestone exhaustive documentation request. |
| 2026-04-22 | Added canonical full-healing plan link under Planning. | The full proposed healing plan now has a durable filesystem copy and project-local KB note, so the project hub needed a direct planning entrypoint. | User request to confirm the entire plan is logged on the filesystem and in the KB. |
| 2026-04-26 | Added live-test failure map and 3000+ word retrospective links under Browser Testing. | A live-test session via Chrome DevTools MCP reached zero of twelve acceptance criteria; the failure catalog needed a hub-level entry point so the next agent doesn't repeat the dead-ends. | Direct user instruction during the wxt-prompt live-test session. |