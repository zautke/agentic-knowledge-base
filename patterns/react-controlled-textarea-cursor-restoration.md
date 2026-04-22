---
title: react-controlled-textarea-cursor-restoration
type: pattern
permalink: agent-kb/patterns/react-controlled-textarea-cursor-restoration
created: 2026-04-04
modified: 2026-04-04
entity_type: pattern
status: evergreen
tags:
- domain/react
- domain/ui
- domain/keyboard
- pattern/cursor
- status/evergreen
aliases:
- cursor restoration
- useLayoutEffect cursor
- controlled textarea cursor fix
---

# React Controlled Textarea Cursor Restoration

## Problem

When a controlled `<textarea>` calls `setValue(newValue)` (e.g. from a keyboard handler), React re-renders the element with the new value and the browser resets the cursor to the **end of the text**. This is the default browser behaviour for programmatic value assignment. The result is a visible cursor jump on every keystroke that mutates the string.

## Pattern

Use a `useRef` sentinel and `useLayoutEffect` to restore the cursor **synchronously after DOM update, before paint**:

```ts
const pendingCursorRef = useRef<{ start: number; end: number } | null>(null);

// In the keyboard handler — record where cursor should land:
pendingCursorRef.current = { start: newPos, end: newPos };
setValue(newValue);   // triggers React re-render (cursor jumps to end)

// In hook body — restore before the browser paints:
useLayoutEffect(() => {
  if (!pendingCursorRef.current || !textareaRef.current) return;
  const { start, end } = pendingCursorRef.current;
  pendingCursorRef.current = null;
  textareaRef.current.setSelectionRange(start, end);
}, [value]);
```

## Why `useLayoutEffect` and not `useEffect`

`useLayoutEffect` fires **synchronously after DOM mutations but before paint**. `useEffect` fires asynchronously — one frame after paint — so the cursor visibly jumps to the end and snaps back, causing a flash. `flushSync` is a third option but adds complexity and degrades render performance.

## Sentinel pattern (`pendingCursorRef`)

The `useLayoutEffect` depends on `value`. Without a sentinel it would fire on **every** value change, including normal typing where the cursor is already correct (React preserves cursor during `onChange`). The `null` sentinel means: "only restore cursor when a programmatic mutation requested it."

## Cursor-only moves (no restoration needed)

When an operation only **moves the cursor** without changing the string (e.g. Ctrl+A to go to line start), skip `setValue` entirely and call `el.setSelectionRange(pos, pos)` directly. React won't re-render, so the cursor sticks without any restoration dance.

## Python / non-React equivalent

prompt_toolkit's `Buffer.set_document(Document(new_text, cursor_position=n))` updates text and cursor **atomically** — this problem does not exist outside React's controlled-component model.

## Observations

- [confirmed: 2026-04-04] Validated in `src/hooks/useReadlineKeys.ts` (wxt-prompt) with 12 readline shortcuts
- [constraint] The sentinel `null` check is required to avoid firing on normal typing; without it every character typed would trigger a `setSelectionRange` call
- [anti-pattern] `useEffect` instead of `useLayoutEffect` causes a one-frame cursor flash — visible at 60fps
- [anti-pattern] `flushSync` works but is discouraged: can cause nested-update errors and hurts perceived performance

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-04-04 | Created | Extracted from wxt-prompt readline implementation session | Bridge Protocol — general React truth should not stay in project folder |

## Relations

- implemented_by [[Readline Keybindings ADR – wxt-prompt]]
- related_to [[readline-keybinding-hook-architecture]]
- related_to [[TUI Implementation Guide]]