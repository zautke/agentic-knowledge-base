---
title: TUI Implementation Guide
type: reference
permalink: projects/blogg/guides/tui-implementation
entity_type: reference
status: evergreen
tags:
- project/blogg
- domain/guides
- domain/tui
---

# TUI Implementation Guide — /list-teams Interactive Browser

## Quick Start

```bash
brew install gum jq       # Dependencies
./team-browser.sh          # Interactive mode
./team-browser.sh --basic  # No-gum fallback
./team-browser.sh --benchmark  # Performance test
```

## What Was Built

A sub-100ms interactive team browser replacing static `/list-teams`:
- Instant header rendering (< 50ms target)
- Lazy-loaded team details (only on selection)
- Interactive navigation with arrow keys
- Smart caching for offline capability
- Graceful degradation (works without gum)

## Performance Results

| Metric | Result | Target | Status |
|--------|--------|--------|--------|
| Cache generation | 49ms | < 100ms | PASS |
| Cache read | 29ms | < 10ms | FAIL |
| Detail fetch | 32ms | < 200ms | PASS |
| Header render | 175ms | < 50ms | FAIL |

## File Structure

```
team-browser.sh                              # Main executable (582 lines)
docs/architecture/TUI_ARCHITECTURE.md        # Design document
docs/guides/GUM_PATTERNS.md                  # Gum reference
docs/guides/TUI_IMPLEMENTATION_GUIDE.md      # This guide (full version)
```

## Relations

- child_of [[Next.js Blog (blogg)]]
- implements [[TUI Team Browser Architecture]]
- uses [[Gum TUI Patterns]]