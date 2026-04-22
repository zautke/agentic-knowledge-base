---
title: CLI Performance Analysis
type: reference
permalink: projects/blogg/analysis/cli-performance
entity_type: reference
status: evergreen
tags:
- project/blogg
- domain/analysis
- domain/performance
---

# CLI Performance Analysis: /list-teams Optimization

**Mission:** Achieve sub-100ms initial render for team list display.
**Baseline:** 2000-3500ms (unacceptable). **Target:** < 100ms initial, < 200ms drill-down.

## Latency Breakdown (Before Optimization)

| Source | Time (ms) | % of Total | Type |
|--------|-----------|------------|------|
| 4x `read_note` calls | 800-1600 | 50-60% | Network I/O (MCP) |
| `switch_project` | 300-500 | 15-20% | Network I/O (MCP) |
| `build_context` | 200-400 | 10-15% | Network I/O (MCP) |
| `list_directory` | 150-250 | 7-10% | Network I/O (MCP) |
| Parse + format | 50-100 | 3-5% | Local CPU |
| Terminal output | 10-20 | < 1% | Local I/O |

**Root cause:** 90%+ of latency is serial MCP network calls. Each `read_note` is ~200-400ms.

## Optimization Strategy

1. **Local caching** — Cache team data to filesystem, read from cache (< 10ms)
2. **Lazy loading** — Only fetch selected team's details on demand
3. **Background refresh** — Update cache asynchronously, serve stale immediately
4. **Progressive disclosure** — Show list first (from cache), details on selection

## Outcome

Led to the [[TUI Team Browser Architecture]] design with gum-based interactive UI achieving near-target performance.

## Relations

- child_of [[Next.js Blog (blogg)]]
- led_to [[TUI Team Browser Architecture]]
- related_to [[Gum TUI Patterns]]
