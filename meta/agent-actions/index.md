---
created: 2026-04-21
modified: 2026-05-28
entity_type: instruction
status: evergreen
title: create-entity
type: instruction
permalink: meta/agent-actions/create-entity
tags:
- system/action
- meta/instruction
- type/create
- source/largo
- machine/largo
---

# Action: Create Entity

## Directive
Create new graph node with proper Entity-Observation-Relation structure.

## Observations
- [precondition] Determine entity_type before creation: note, concept, decision, project, reference, meeting, person
- [precondition] Identify target folder using 2-3 level hierarchy max
- [structure] Title must follow naming convention for entity_type
- [structure] Include minimum 2-3 observations with category annotations
- [structure] Include at minimum 1 relation to existing entity for graph connectivity
- [validation] Verify no duplicate permalink exists via search_notes before creation

## Execution Pattern
```
1. search_notes(query="{proposed_title}") → check duplicates
2. list_directory(dir_name="{target_folder}") → verify location
3. write_note(
     folder="{path}",
     title="{Title}",
     content="{structured_markdown}",
     entity_type="{type}",
     tags=["{namespace/tag}"]
   )
4. read_note(identifier="{permalink}") → verify creation
```

## Content Template
```markdown
# {Title}

## Observations
- [{category}] {atomic_fact} #{optional_tag}
- [{category}] {atomic_fact}

## Relations
RELATION_LINE: {relation_type} [[{Existing Entity}]]
```

## Relations
- part_of [[meta/agent-actions/index]]
- implements [[Entity Creation Protocol]]
