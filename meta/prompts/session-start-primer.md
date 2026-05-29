---
title: session-start-primer
created: 2026-01-26
modified: 2026-05-28
type: meta_prompt
entity_type: meta_prompt
status: evergreen
permalink: meta/prompts/session-start-primer
tags:
- system/meta-prompt
- meta/session
- type/primer
- system/core
- source/largo
- machine/largo
---

# SessionStart: Agent Knowledge Base Primer

## Directive
Context injection for AI agents at session initialization. Execute this sequence at conversation start when operating with Basic-Memory MCP.

## Observations
- [purpose] Establish shared context between agent and knowledge graph
- [trigger] Execute on: session start, project switch, or explicit context refresh request
- [efficiency] Retrieves minimal context for orientation; expand on-demand
- [idempotent] Safe to re-execute; refreshes rather than duplicates context
- [requirement] Session start must include a capability and maintenance snapshot before semantic curation begins
- [requirement] Agents must know whether semantic search is enabled and whether embeddings are current

## Initialization Sequence

### Step 1: Orient (Required)
```
build_context(
  url="memory://meta/readme",
  depth=1,
  timeframe="30d"
)
```
**Purpose**: Load system architecture, tool reference, and core relations.

### Step 2: Load Actions (Recommended)
```
build_context(
  url="memory://meta/agent-actions/*",
  depth=1
)
```
**Purpose**: Load available action instruction patterns.

### Step 3: Assess Recent Activity (Situational)
```
recent_activity(
  timeframe="7d",
  depth=1,
  max_related=5
)
```
**Purpose**: Understand recent knowledge evolution and active work context.

### Step 4: Capability Snapshot (Required For Curation)
```
project_info(project="{active_project}") → semantic search status, embedding coverage, unresolved count, isolated count
```
**Purpose**: Determine whether semantic retrieval is trustworthy enough for linking/tagging work and whether maintenance is required first.

### Step 5: Schema and Maintenance Gate (Required For Taxonomy / Type Work)
```
schema_diff(note_type="{dominant_type}") → current drift
schema_validate() → existing validation surface
```
**Purpose**: Determine whether the session should include schema normalization, note-type cleanup, or reindexing.

### Step 6: Task-Specific Context (On-Demand)
```
search_notes(query="{task_keywords}") → initial candidate set
search_notes(query="{task_keywords}", search_type="hybrid") → semantic candidate set when enabled
search_notes(tags=["domain/{subject}"]) → tag-neighborhood candidates
build_context(url="memory://{relevant_entity}", depth=2) → deep context
```
**Purpose**: Expand context for specific task requirements.

## Agent Operational Guidelines

### DO
- Read meta/README before any knowledge operations
- Follow action instructions in meta/agent-actions/
- Use established taxonomy from meta/taxonomy/index
- Create relations to existing entities
- Use namespaced tags
- Verify operations via build_context or read_note
- Check project embedding status before semantic curation
- Use both lexical and semantic retrieval when semantic search is current
- Use schema tools when curation changes note classes, relation vocabulary, or
  folder taxonomy
- Report exact structural changes and exact discovery steps to the user

### DO NOT
- Create orphan entities without relations
- Invent new tag namespaces without updating taxonomy
- Nest folders beyond 3 levels
- Duplicate content; use relations instead
- Skip verification after write operations
- Trust semantic ranking if embeddings are stale or missing
- Close a KB task with vague statements like "updated links" or "tagged related notes"

## Deterministic Curation Gate

Before any linking or tagging task, complete this gate in order:
1. Capability snapshot.
2. Schema and maintenance gate.
3. Neighborhood discovery via search and context traversal.
4. Taxonomy decision.
5. Structural update.
6. Verification.
7. Exact user-facing observability report.

## Quick Reference: Core Tools
| Action | Tool | Minimal Call |
|--------|------|--------------|
| Read entity | read_note | `read_note(identifier="{title_or_permalink}")` |
| Create entity | write_note | `write_note(folder, title, content, entity_type)` |
| Update entity | edit_note | `edit_note(identifier, operation, content)` |
| Search | search_notes | `search_notes(query="{keywords}")` |
| Traverse | build_context | `build_context(url="memory://{path}")` |
| Recent | recent_activity | `recent_activity(timeframe="7d")` |
| List | list_directory | `list_directory(dir_name="/")` |

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-05-28 | Fixed malformed tag indentation (`source/largo`, `machine/largo` were over-indented under `system/core`); added missing frontmatter fields (`created`, `modified`, `entity_type`, `status`). | Tags rendered as nested children rather than top-level list items, breaking tag-neighborhood discovery; frontmatter was missing 4 of 7 required contract fields. | KB hygiene sweep. |

## Relations
- part_of [[meta/README]]
- uses [[meta/agent-actions/index]]
- uses [[meta/taxonomy/index]]
- implements [[Agent Session Protocol]]
- related_to [[Curator Session Primer and Runbook]]
