---
created: 2026-04-21
modified: 2026-05-28
entity_type: meta_prompt
status: evergreen
title: bootstrap-knowledge-base
type: meta_prompt
permalink: meta/prompts/bootstrap-knowledge-base
tags:
- system/meta-prompt
- meta/bootstrap
- type/instruction
- source/largo
- machine/largo
---

# Meta-Prompt: Bootstrap Knowledge Base

## Directive
Initialize a new Basic-Memory project with foundational structure and core documentation.

## Observations
- [purpose] This prompt bootstraps an empty project into an agent-navigable knowledge base
- [idempotent] Safe to re-run; checks for existing entities before creation
- [prerequisite] Project must be created and switched to via switch_project
- [output] Creates: README, action index, taxonomy index, folder structure

## Execution Sequence

### Phase 1: Verify State
```
1. get_current_project() → confirm target project active
2. list_directory(dir_name="/", depth=1) → verify empty or assess existing state
3. IF entities exist: build_context(url="memory://meta/*", depth=1) → assess completeness
```

### Phase 2: Create Core Structure
```
1. write_note(
     folder="meta",
     title="README", 
     entity_type="knowledge-base",
     content="{README_TEMPLATE}",
     tags=["system/core", "meta/bootstrap"]
   )

2. write_note(
     folder="meta/agent-actions",
     title="index",
     entity_type="index",
     content="{ACTION_INDEX_TEMPLATE}"
   )

3. write_note(
     folder="meta/taxonomy", 
     title="index",
     entity_type="index",
     content="{TAXONOMY_TEMPLATE}"
   )
```

### Phase 3: Create Action Instructions
```
For each action in [create-entity, link-entities, search-graph, traverse-context, update-entity, manage-taxonomy]:
  write_note(
    folder="meta/agent-actions",
    title="{action-name}",
    entity_type="instruction",
    content="{ACTION_TEMPLATE}"
  )
```

### Phase 4: Verify Graph Connectivity
```
1. build_context(url="memory://meta/readme", depth=2) → verify relations
2. list_directory(dir_name="meta", depth=2) → confirm structure
3. search_notes(query="system/core") → verify tag indexing
```

## Success Criteria
- [ ] meta/README.md exists with observations and relations
- [ ] meta/agent-actions/index.md links to all action instructions
- [ ] meta/taxonomy/index.md defines folder/tag conventions
- [ ] All action instructions are created and linked
- [ ] build_context returns connected graph from README

## Relations
- part_of [[meta/agent-actions/index]]
- implements [[Bootstrap Protocol]]
- successor_of [[Empty Project State]]
