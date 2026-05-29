---
created: 2026-04-21
modified: 2026-05-28
entity_type: index
status: evergreen
title: index
type: index
permalink: meta/prompts/index
tags:
  - system/core
  - meta/prompts
  - type/index
  - source/largo
  - machine/largo
---

# Meta-Prompts Index

## Purpose
Registry of agent-instructing meta-prompts for autonomous knowledge base operations.

## Observations
- [design] Meta-prompts are "agent teaching agent" instructions for idempotent operations
- [hierarchy] Prompts execute in dependency order: bootstrap → extend → session
- [token-efficiency] Each prompt designed for minimal context retrieval before execution

## Prompt Registry

### Lifecycle Prompts
| Prompt | Purpose | When to Use |
|--------|---------|-------------|
| [[meta/prompts/bootstrap-knowledge-base]] | Initialize empty project | New project setup |
| [[meta/prompts/extend-knowledge-graph]] | Add entities with proper taxonomy | Creating new content |
| [[meta/prompts/session-start-primer]] | Agent context injection | Session initialization |

### Execution Dependencies
```
bootstrap-knowledge-base
    └── Creates: README, action-index, taxonomy-index
           │
           ▼
extend-knowledge-graph  
    └── Requires: README context, taxonomy conventions
           │
           ▼
session-start-primer
    └── Requires: All above exist; used at runtime
```

## Relations
- part_of [[meta/README]]
- contains [[meta/prompts/bootstrap-knowledge-base]]
- contains [[meta/prompts/extend-knowledge-graph]]
- contains [[meta/prompts/session-start-primer]]