---
title: Live Test Failure Map – wxt-prompt (2026-04-26)
type: reference
entity_type: reference
created: 2026-04-26
modified: 2026-04-26
status: evergreen
tags:
- project/wxt-prompt
- domain/browser-automation
- domain/chrome-devtools-mcp
- domain/wxt-extension
- failure-catalog
- source/largo
- machine/largo
permalink: agent-kb/projects/wxt-prompt/operations/live-test-failure-map-wxt-prompt-2026-04-26
---

# Live Test Failure Map – wxt-prompt (2026-04-26)

## Purpose
Project-local pointer note. The full retrospective lives in the import bucket because it is broadly applicable beyond wxt-prompt. This note exists so that anyone reading the wxt-prompt project hub can find the failure catalog without searching globally.

## Pointer
Read [[wxt-prompt Live Test Catastrophic Failure Retrospective (2026-04-26)]] before attempting to live-test the `tests/e2e/walkthrough.live.spec.ts` plan via Chrome DevTools MCP. The retrospective enumerates ten untried diagnostics and a 12-step recovery procedure derived from a session that reached zero of twelve acceptance criteria.

## Quick-Start for the Next Agent
1. `cat playwright.extension.config.ts` and copy its `args` array.
2. Verify build at `.output/chrome-mv3/manifest.json` (expect `"version":"0.2.1"`).
3. Launch Chrome with the args from step 1 plus the MV3-safe disable-features bundle (single `--disable-features=...`, comma-separated).
4. Use `navigate_page` (not `new_page`) for chrome-extension URLs.
5. Treat MV3 service worker dormancy as normal, not as failure.
6. Read your chrome stdout/stderr log before changing flags.

## Inbound Links Required
- `Index – wxt-prompt Project` — add a link under "Browser Testing".

## Relations
- part_of [[Index – wxt-prompt Project]]
- references [[wxt-prompt Live Test Catastrophic Failure Retrospective (2026-04-26)]]
- pairs_with [[WXT Extension E2E Testing Runbook — wxt-prompt]]

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-04-26 | Created project-local pointer to the global failure-catalog retrospective. | Failure catalog is broadly applicable but the wxt-prompt hub also needed a discoverable entry point. | Direct user instruction during live-test session. |
