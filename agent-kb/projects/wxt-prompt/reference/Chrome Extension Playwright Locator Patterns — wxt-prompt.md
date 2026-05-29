---
title: Chrome Extension Playwright Locator Patterns — wxt-prompt
type: reference
permalink: agent-kb/projects/wxt-prompt/reference/chrome-extension-playwright-locator-patterns-wxt-prompt
created: 2026-04-21
modified: 2026-04-21
entity_type: reference
status: evergreen
tags:
- project/wxt-prompt
- domain/playwright
- domain/browser-extension
- domain/testing
- status/evergreen
- source/largo
- machine/largo
---

# Chrome Extension Playwright Locator Patterns — wxt-prompt

## Purpose

Verified CSS selectors and Playwright locator patterns for the wxt-prompt sidepanel UI. Corrects known bad locators and provides the authoritative working ones.

## Verified Locators

### Header Controls

| Element | Locator | Notes |
|---------|---------|-------|
| Hamburger / conversations | `page.getByLabel('Open conversations')` | aria-label on button |
| DevTools wrench toggle | `page.getByLabel('Toggle DevTools mode')` | aria-label on button |
| Settings gear | `page.getByLabel('Open settings')` | aria-label on button |

### DevTools Toggle State

```typescript
// Check if DevTools mode is ON
const isOn = await wrench.evaluate(
  (el) => el.classList.contains('text-yellow-600') || el.classList.contains('bg-yellow-50'),
);

// Assert ON
await expect(wrench).toHaveClass(/text-yellow-600/, { timeout: 3_000 });
// Assert OFF
await expect(wrench).not.toHaveClass(/text-yellow-600/, { timeout: 3_000 });
```

### Chat Input

| Element | Locator |
|---------|---------|
| Message textarea | `page.getByLabel('Chat input')` |
| Send button | `page.getByLabel('Send message')` |
| Send keyboard shortcut | `await input.press('Control+Enter')` |

### Model Picker

| State | Locator |
|-------|---------|
| Connected — select present | `page.getByLabel('Select model')` |
| Disconnected — fallback text | `page.getByText(/No connection — configure a provider/i)` |
| Model options | `picker.locator('option').evaluateAll(...)` |

### Message Bubbles

| Element | Locator | Notes |
|---------|---------|-------|
| Assistant message | `page.locator('li:has(.items-start)').last()` | ✅ Correct |
| Assistant message | `page.locator('li.justify-start').last()` | ❌ Wrong — class on child, not `<li>` |
| User message | `page.locator('li:has(.items-end)').last()` | Follow same pattern |

**The `li.justify-start` trap**: In `ChatPane`, message bubbles render as `<li>` elements. The alignment class (`items-start` for assistant, `items-end` for user) is applied to an inner flex container, not the `<li>` itself. Always use `:has()` to match on the child.

### Streaming State

```typescript
// Streaming in progress — send button hidden/disabled
await expect(page.getByLabel('Send message')).not.toBeVisible();

// Streaming complete — send button visible
await expect(page.getByLabel('Send message')).toBeVisible({ timeout: 60_000 });
```

## Sidepanel URL Pattern

```typescript
// Open sidepanel directly (preferred over clicking extension icon)
await page.goto(`chrome-extension://${extensionId}/sidepanel.html`);
// Always wait for hamburger to confirm React mounted
await expect(page.getByLabel('Open conversations')).toBeVisible({ timeout: 15_000 });
```

## Error Detection Pattern

```typescript
const errors: string[] = [];
page.on('pageerror', (err) => errors.push(err.message));
page.on('console', (msg) => {
  if (msg.type() === 'error') errors.push(msg.text());
});
// Load page...
const fatalErrors = errors.filter(
  (e) =>
    !e.includes('favicon') &&
    !e.includes('ERR_FILE_NOT_FOUND') &&
    !e.includes('net::') &&
    !e.includes('Failed to fetch'),
);
expect(fatalErrors).toHaveLength(0);
```

The filter removes browser-generated noise. Only extension-originated errors count.

## Relations
- part_of [[Index – wxt-prompt Project]]
- related_to [[WXT Extension E2E Testing Runbook — wxt-prompt]]
- related_to [[Injectable Provider Tool Loop Pattern – wxt-prompt]]

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------| 
| 2026-04-21 | Created. Verified locators from 2026-04-21 milestone.live.spec.ts run. Documents the `li.justify-start` vs `li:has(.items-start)` bug. | Future agents need verified selectors before writing extension tests. | User t+2 designation and exhaustive KB documentation request. |