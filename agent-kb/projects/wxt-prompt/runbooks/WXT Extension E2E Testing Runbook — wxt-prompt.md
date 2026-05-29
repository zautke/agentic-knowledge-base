---
title: WXT Extension E2E Testing Runbook — wxt-prompt
type: instruction
permalink: agent-kb/projects/wxt-prompt/runbooks/wxt-extension-e2-e-testing-runbook-wxt-prompt
created: 2026-04-21
modified: 2026-04-21
entity_type: instruction
status: evergreen
tags:
- project/wxt-prompt
- domain/browser-extension
- domain/playwright
- domain/testing
- op/create-node
- status/evergreen
- source/largo
- machine/largo
---

# WXT Extension E2E Testing Runbook — wxt-prompt

## Purpose

Step-by-step runbook for running Playwright E2E tests against the wxt-prompt Chrome MV3 extension. Written from the lessons of the 2026-04-21 milestone browser verification run.

## Prerequisites

- Node.js with `pnpm` available
- Playwright installed: `pnpm install` in the project root
- Extension built: `pnpm build` produces `.output/chrome-mv3/`

## Build Verification (do this first)

```bash
ls .output/chrome-mv3/manifest.json    # must exist
ls .output/chrome-mv3/sidepanel.html   # must exist
```

If missing: run `pnpm build` from the worktree directory (not the project root — they diverge in a worktree).

## Run the Milestone Test Suite

```bash
npx playwright test \
  --config playwright.extension.config.ts \
  tests/e2e/milestone.live.spec.ts
```

Scenarios 1-3 pass without any external services. Scenarios 4-5 skip gracefully if Ollama is unavailable.

## Run All E2E Tests

```bash
npx playwright test --config playwright.extension.config.ts
```

## Interpreting Results

| Result | Meaning |
|--------|---------|
| PASS | Verified |
| SKIP (Ollama not available) | Correct — provider-gated test, not a failure |
| FAIL (extension load) | Build is broken or wrong path |
| FAIL (UI render) | React app broken |
| FAIL (DevTools persistence) | `usePersistentFlag` or Dexie broken |

## Critical Configuration Facts

| Fact | Value |
|------|-------|
| Build output path | `.output/chrome-mv3/` (not `dist/`) |
| Extension ID | Discovered dynamically from service worker URL — never hardcode |
| Playwright version | 1.58.2 (2026-04-21 run) |
| Chromium version | 145 (2026-04-21 run) |
| `launchPersistentContext` | Required — `launch()` won't register service worker |
| `--disable-extensions-except` | Required — prevents Chrome default extensions from polluting SW list |

## Ollama Connectivity

| Endpoint | Connect Time | Notes |
|----------|-------------|-------|
| `https://ollama.braisenly.com` | ~5 seconds | Use for automated / CI tests |
| `http://localhost:11434` | 60+ seconds | May not be reachable from headless sandbox |

Poll for up to 180 seconds before declaring Ollama unavailable. The `isOllamaAvailable()` helper in `milestone.live.spec.ts` does a 5-second check — wrap it in a retry loop for automated runs.

## Known Locator Patterns

| Selector | Correct? | Notes |
|----------|----------|-------|
| `li:has(.items-start)` | ✅ Yes | Assistant message bubbles |
| `li.justify-start` | ❌ No | Class is on a child element, not the `<li>` itself |
| `[aria-label="Open conversations"]` | ✅ Yes | Hamburger button |
| `[aria-label="Toggle DevTools mode"]` | ✅ Yes | Wrench button |
| `[aria-label="Open settings"]` | ✅ Yes | Gear button |
| `[aria-label="Chat input"]` | ✅ Yes | Textarea input |
| `[aria-label="Send message"]` | ✅ Yes | Send button (visible when not streaming) |

## Test-Layer Strategy

Design tests in three layers:

1. **Structural** (no network): extension load, UI render, state persistence — always runs
2. **Provider-dependent**: requires live LLM — guard with `isOllamaAvailable()` + skip
3. **Tool-call**: requires live LLM + specific model + DevTools ON — guard with `test.skip()`

Never report a skipped provider test as a failure.

## Relations
- part_of [[Index – wxt-prompt Project]]
- related_to [[Injectable Provider Tool Loop Pattern – wxt-prompt]]
- related_to [[Ollama Provider Connection Timing Reference — wxt-prompt]]
- related_to [[Chrome Extension Playwright Locator Patterns — wxt-prompt]]
- related_to [[browser-extension-cdp-testing-patterns]]

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------| 
| 2026-04-21 | Created. Distilled from the 2026-04-21 milestone browser verification run. | Runbook captures all hard-won operational facts so future agents do not repeat the same discovery process. | User t+2 designation and exhaustive KB documentation request. |