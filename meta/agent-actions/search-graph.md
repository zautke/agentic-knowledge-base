---
created: 2026-04-21
modified: 2026-05-28
entity_type: instruction
status: evergreen
title: traverse-context
type: instruction
permalink: meta/agent-actions/traverse-context
tags:
  - system/action
  - meta/instruction
  - type/traverse
  - source/largo
  - machine/largo
---

# Action: Traverse Context

## Directive
Execute depth-controlled graph traversal to build contextual understanding from relations.

## Observations
- [mechanism] build_context walks directed edges from root entity outward
- [parameter] depth=1 returns immediate neighbors only (fastest, minimal context)
- [parameter] depth=2 explores two relation levels (recommended for contextual inference)
- [parameter] depth=3+ use sparingly; exponential expansion risk
- [parameter] timeframe filters by modification recency: "7d", "last week", "2 days ago"
- [parameter] max_related caps returned entities per depth level (default 10)
- [optimization] Use depth=1 for existence checks; depth=2 for comprehension
- [output] Returns root entity, related entities with distance metrics, connection paths

## Execution Pattern
```
build_context(
  url="memory://{permalink_or_pattern}",
  depth=2,
  timeframe="7d",
  max_related=10
)
```

## URI Pattern Strategies
| Pattern | Use Case | Example |
|---------|----------|---------|
| Direct | Known entity | `memory://meta/readme` |
| Folder wildcard | All in folder | `memory://projects/*` |
| Prefix match | Category scan | `memory://decision*` |
| Suffix match | Type discovery | `memory://*/index` |

## Traversal Decision Tree
```
Need specific entity? → depth=1, direct URL
Need related context? → depth=2, direct URL  
Need category overview? → depth=1, wildcard pattern
Building comprehensive context? → depth=2, iterative expansion
```

## Relations
- part_of [[meta/agent-actions/index]]
- related_to [[meta/agent-actions/search-graph]]
- implements [[Graph Traversal Protocol]]