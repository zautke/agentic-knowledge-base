---
title: readline-keybinding-hook-architecture
type: pattern
permalink: agent-kb/patterns/readline-keybinding-hook-architecture
created: 2026-04-04
modified: 2026-04-04
entity_type: pattern
status: evergreen
tags:
- domain/keyboard
- domain/react
- domain/ux
- pattern/architecture
- status/evergreen
aliases:
- readline hook
- emacs keybindings react
- kill ring hook
---

# Readline Keybinding Hook Architecture

## Problem

No npm package (as of April 2026, confirmed by search) provides readline/emacs-style text manipulation shortcuts for web `<textarea>` elements. Adding these shortcuts to a React controlled textarea requires:
1. Direct access to `selectionStart` / `selectionEnd` (not available from a hotkey library callback)
2. Text mutation that plays correctly with React's controlled-component model
3. Cursor position restoration after `setState`
4. A kill ring for Ctrl+K / Ctrl+U / Ctrl+W / Alt+D → Ctrl+Y roundtrip

## Two-Layer Architecture

Split keyboard concerns into two independent layers:

| Layer | What it owns | Implementation |
|-------|-------------|----------------|
| **Text manipulation hook** | String mutation, cursor, kill ring | `useReadlineKeys` custom hook |
| **Semantic dispatch** | Submit, history nav, stub callbacks | `InputBarKeyboardHandler` render-prop component |

These layers are intentionally decoupled. The hook can be swapped or tested without touching semantic dispatch. New semantic callbacks can be added without touching text-manipulation logic.

### Composition pattern

```tsx
const readlineOnKeyDown = useReadlineKeys(textareaRef, value, setValue);

<textarea
  onKeyDown={(e) => {
    readlineOnKeyDown(e);                      // text layer first
    if (!e.defaultPrevented) existingOnKeyDown(e); // semantic layer second
  }}
/>
```

The `defaultPrevented` guard is the handshake: readline sets it when it handles a key; semantic layer skips it.

## Hook internals

All hook state is `useRef` (zero re-renders):

| Ref | Purpose |
|-----|---------|
| `killRingRef` | Single-entry kill buffer (string) |
| `lastWasKillRef` | True when previous key was a kill op — consecutive kills accumulate |
| `pendingCursorRef` | Cursor position to restore after React re-render (see [[react-controlled-textarea-cursor-restoration]]) |

## Word boundary algorithm (readline semantics)

`\w` = alphanumeric + underscore. Skip non-word chars, then skip word chars. Mirrors GNU readline's `backward-kill-word` / `forward-word` exactly.

```ts
function wordBoundaryBack(text, pos) {
  while (pos > 0 && !/\w/.test(text[pos - 1])) pos--;
  while (pos > 0 && /\w/.test(text[pos - 1])) pos--;
  return pos;
}
```

## Shortcuts implemented

| Binding | Action | Notes |
|---------|--------|-------|
| Ctrl+K | Kill cursor → EOL | Consecutive kills append |
| Ctrl+U | Kill BOL → cursor | |
| Ctrl+W | Kill word backward | Capturable in Chrome sidepanel via preventDefault |
| Ctrl+D | Delete char forward | |
| Ctrl+H | Delete char backward | |
| Ctrl+Y | Yank from kill ring | |
| Ctrl+A | Cursor to line start | Overrides browser "select all" |
| Ctrl+E | Cursor to line end | |
| Ctrl+B | Cursor back one char | |
| Alt+B | Cursor back one word | |
| Alt+F | Cursor forward one word | |
| Alt+D | Kill word forward | |

**Skipped** (browser intercepts before page even in sidepanel): Ctrl+T, Ctrl+F

**Ctrl+W note:** Chrome normally intercepts Ctrl+W to close the tab, but in an extension sidepanel context `preventDefault()` in the page's `keydown` handler suppresses it. Confirmed Chrome 120+.

## Stub callback layer (semantic dispatch)

The `InputBarKeyboardHandler` component accepts optional callbacks for keys that need extensibility hooks but no current behaviour:

```ts
onTab?, onArrowLeft?, onArrowRight?,
onCtrlEnter?, onShiftEnter?, onAltEnter?, onEscape?
```

Each defaults to a no-op. Future features attach here without modifying the keyboard layer.

## Python port

Ports cleanly to `prompt_toolkit.KeyBindings`:
- `Buffer.set_document(Document(text, cursor_position=n))` replaces `setValue` + cursor restoration (atomic, no React model)
- Alt+B/F/D become `('escape', 'b/f/d')` two-key sequences (terminals send ESC + char, not a Web API `altKey`)
- `c-m` (Enter) replaces Ctrl+Enter (terminals cannot distinguish)

## Observations

- [confirmed: 2026-04-04] No npm package for web textarea readline keybindings found (April 2026 web search)
- [confirmed: 2026-04-04] react-hotkeys-hook rejected: cannot access `selectionStart`/`selectionEnd` in its callback scope without a ref — would require 12+ separate hook calls with ref threading
- [constraint] `useRef` for all hook state — none of kill ring / cursor / kill-flag state should trigger re-renders
- [constraint] "Line" = logical line delimited by `\n`, not visual wrapped line
- [anti-pattern] Putting readline logic in the semantic dispatch component — they have different dependency surfaces (one needs the textarea ref and value state; the other needs streaming/disabled flags and semantic callbacks)
- [source] GNU readline emacs mode: https://readline.kablamo.org/emacs.html
- [source] April 2026 web search confirming no dedicated npm package

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-04-04 | Created | Extracted from wxt-prompt readline implementation | Bridge Protocol — architecture pattern is globally reusable |

## Relations

- implemented_by [[Readline Keybindings ADR – wxt-prompt]]
- depends_on [[react-controlled-textarea-cursor-restoration]]
- related_to [[TUI Implementation Guide]]