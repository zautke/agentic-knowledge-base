---
title: Ollama Provider Connection Timing Reference — wxt-prompt
type: reference
permalink: agent-kb/projects/wxt-prompt/reference/ollama-provider-connection-timing-reference-wxt-prompt
created: 2026-04-21
modified: 2026-04-21
entity_type: reference
status: evergreen
tags:
- project/wxt-prompt
- domain/ollama
- domain/browser-extension
- domain/testing
- status/evergreen
- source/largo
- machine/largo
---

# Ollama Provider Connection Timing Reference — wxt-prompt

## Purpose

Empirically measured Ollama connection times for the wxt-prompt extension across different endpoint types. Written so future agents do not timeout too early or wait unnecessarily long.

## Measured Connection Times

| Endpoint Type | Endpoint | Connect Time | Context |
|--------------|----------|-------------|---------|
| Remote HTTPS | `https://ollama.braisenly.com` | **~5 seconds** | 2026-04-21 milestone run, headless Chromium 145 |
| Local HTTP | `http://localhost:11434` | **60+ seconds** | Typical cold start in extension context |
| Local HTTP (headless sandbox) | `http://localhost:11434` | **Unreachable** | Some headless configs block localhost from within Chrome process |

## Recommended Poll Window

- Remote endpoint: 30 seconds is generous
- Local endpoint: 180 seconds minimum before declaring unavailable
- Headless sandbox with local Ollama: assume unavailable unless explicitly configured

## How the Extension Connects

1. On mount, `useProviders` fires `probeProvider()` for each configured provider
2. The probe makes a `GET /api/version` or `/api/models` request from the service worker
3. The UI updates `connectionState` to `'connected'` when the probe succeeds
4. The `[aria-label="Connected"]` indicator appears in the sidebar/header

## Test-Side Detection

In Playwright tests, detect connection via:

```typescript
// Option 1: Check Ollama API from browser context (most reliable)
const response = await context.request.get('http://localhost:11434/api/version', { timeout: 5_000 });
return response.ok();

// Option 2: Wait for connected indicator in UI (requires provider configured in settings)
await page.waitForSelector('[aria-label="Connected"]', { timeout: 180_000 });

// Option 3: Check model picker populated (implies connected + models loaded)
const select = page.getByLabel('Select model');
await select.waitFor({ state: 'attached', timeout: 180_000 });
```

## Why This Matters

During the 2026-04-21 milestone session, the browser test agent was initially dispatched with `localhost:11434`. The agent hung silently for 10+ minutes waiting for a provider that was unreachable in the headless sandbox. Switching to `https://ollama.braisenly.com` resulted in connection in 5 seconds.

Always use the remote endpoint for automated tests. Document the active remote endpoint prominently in any test dispatch prompt.

## Active Remote Endpoint

`https://ollama.braisenly.com` — verified working 2026-04-21. Model available: `deepseek-v3.1:671b-clo` (function-calling capable).

## Relations
- part_of [[Index – wxt-prompt Project]]
- related_to [[WXT Extension E2E Testing Runbook — wxt-prompt]]
- related_to [[Injectable Provider Tool Loop Pattern – wxt-prompt]]

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------| 
| 2026-04-21 | Created. Empirical timing data from 2026-04-21 milestone browser verification run. | Remote endpoint connected in 5s; local timeout was causing test agent to hang silently. | User t+2 designation and exhaustive KB documentation request. |