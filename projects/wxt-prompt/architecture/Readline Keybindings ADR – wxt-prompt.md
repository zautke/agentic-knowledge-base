---
title: Readline Keybindings ADR – wxt-prompt
type: reference
permalink: agent-kb/projects/wxt-prompt/architecture/readline-keybindings-adr-wxt-prompt
created: 2026-04-04
modified: 2026-04-04
entity_type: reference
status: evergreen
tags:
- project/wxt-prompt
- domain/keyboard
- domain/react
- op/create-node
- status/evergreen
---

# Readline Keybindings ADR – wxt-prompt

Architecture decision record for adding readline/emacs-style keybindings and extensibility stub callbacks to the InputBar textarea in the `wxt-prompt` Chrome extension sidepanel.

## Context

The `InputBar` component (`src/components/InputBar.tsx`) uses a controlled `<textarea>`. It had no readline-style text manipulation shortcuts. Power users expect Ctrl+K/U/W/Y/A/E and word-movement bindings as muscle memory from terminal and IDE environments.

A secondary need: extensibility hooks on Tab, arrow keys, and Enter variants so future features (tab completion, multi-modal input, etc.) can attach behaviour without re-opening the keyboard layer.

## Decision

### Text manipulation: custom `useReadlineKeys` hook

**Rejected:** `react-hotkeys-hook` — confirmed April 2026, cannot access `selectionStart`/`selectionEnd` inside its callback scope without threading a ref; would require 12+ `useHotkeys` calls; no npm package for readline keybindings on web textareas exists.

**Chosen:** Custom hook `src/hooks/useReadlineKeys.ts` — returns an `onKeyDown` handler, owns kill ring and cursor state internally via `useRef`, uses `useLayoutEffect` for cursor restoration. See [[readline-keybinding-hook-architecture]] and [[react-controlled-textarea-cursor-restoration]].

### Semantic dispatch: optional props on `InputBarKeyboardHandler`

Added 7 optional stub callback props (`onTab`, `onArrowLeft`, `onArrowRight`, `onCtrlEnter`, `onShiftEnter`, `onAltEnter`, `onEscape`) to the existing render-prop component. Each defaults to no-op. The existing props (`onSubmit`, `onHistoryUp`, `onHistoryDown`, `onCancelHistoryNavigation`) are unchanged.

### Handler composition

```tsx
onKeyDown={(e) => {
  readlineOnKeyDown(e);               // readline layer first
  if (!e.defaultPrevented) existingOnKeyDown(e);  // semantic layer second
}}
```

`defaultPrevented` is the handshake. `InputBarKeyboardHandler.handleKeyDown` now opens with `if (event.defaultPrevented) return;` as a defensive guard.

### Ctrl+W

Included despite Chrome normally intercepting it for close-tab. In an extension sidepanel, `preventDefault()` in the page's `keydown` handler suppresses the browser action. Confirmed Chrome 120+.

### Ctrl+T, Ctrl+F

Excluded — browser intercepts before the page even in a sidepanel.

## Files Changed

| File | Change |
|------|--------|
| `src/hooks/useReadlineKeys.ts` | **New** — full readline hook |
| `src/components/InputBarKeyboardHandler.tsx` | 7 optional stub props + `defaultPrevented` guard |
| `src/components/InputBar.tsx` | Import + call hook; compose `onKeyDown`; pass stub no-ops |
| `docs/keyboard-shortcuts.md` | **New** — cheatsheet |

## Python Port

An identical port was written to `python/` using `prompt_toolkit`:

| TS | Python |
|----|--------|
| `useReadlineKeys` hook | `ReadlineKeys` class + `KeyBindings` |
| `InputBarKeyboardHandler` | `InputBarKeyboardHandler` class |
| `useLayoutEffect` cursor restore | `Buffer.set_document()` — atomic, not needed |
| `event.altKey` | `('escape', 'b')` two-key sequence |
| `c-m` distinguish from Ctrl+Enter | Cannot distinguish in terminal; `c-m` = submit |

External reference: `dev-knowledge/input-keyboard-shortcuts/` (3 notes: cheatsheet, implementation, consumer-integration).

## Branch / Commit

- Branch: `development`
- Commit: `c2e8dd6` → rebased to `c7ea880` after pull
- Date: 2026-04-04

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-04-04 | Created | Initial implementation of readline keybindings | User request to add Ctrl+K/U/W and stub callbacks to InputBar |

## Relations

- implements [[readline-keybinding-hook-architecture]]
- implements [[react-controlled-textarea-cursor-restoration]]
- part_of [[Index – wxt-prompt Project]]