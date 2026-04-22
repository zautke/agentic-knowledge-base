---
title: TUI Team Browser Architecture
type: reference
permalink: projects/blogg/architecture/tui-team-browser
entity_type: reference
status: evergreen
tags:
- project/blogg
- domain/architecture
- domain/tui
---

# TUI Team Browser Architecture (TUI-001)

Design document for a gum-based interactive TUI replacing the static `/list-teams` command. Prioritizes sub-100ms initial render, lazy loading, and progressive disclosure.

## Performance Targets

| Operation | Target |
|-----------|--------|
| Initial render | < 100ms |
| Team list display | < 50ms |
| Detail loading | < 200ms |
| Action menu | < 10ms |

## Interaction Phases

1. **Instant Render** (< 100ms): Header + team list from cache
2. **Lazy Load Details** (< 200ms): Fetch only selected team's data with `gum spin`
3. **Detail View**: Formatted display via `gum style --border rounded`
4. **Action Menu**: Spawn, view roster, return, exit
5. **Mission Input**: `gum input` with validation (min 10 chars)
6. **Confirmation**: `gum confirm --default=false`

## Caching Strategy

```
~/.cache/team-browser/
  .count           # 4 bytes, team count
  .list_cache      # ~2KB, team IDs + names
  .last_sync       # 8 bytes, timestamp
  details/         # ~5KB per team, on-demand
```

- List cache TTL: 10 minutes (background refresh)
- Detail cache TTL: 5 minutes (lazy load on selection)
- Graceful degradation: works without gum (basic `nl` output)

## Key Gum Components Used

| Component | Command | Use |
|-----------|---------|-----|
| Header | `gum style --border double` | Styled info banner |
| Team List | `gum choose` / `gum filter` | Interactive selection |
| Details | `gum style --border rounded` | Formatted display |
| Loading | `gum spin --spinner dot` | Progress feedback |
| Input | `gum input --char-limit 500` | Mission entry |
| Confirm | `gum confirm` | Safe action gate |

## Implementation Status

Design complete. Main script: `team-browser.sh` (582 lines).

## Relations

- child_of [[Next.js Blog (blogg)]]
- uses [[Gum TUI Patterns]]
- derived_from [[CLI Performance Analysis]]