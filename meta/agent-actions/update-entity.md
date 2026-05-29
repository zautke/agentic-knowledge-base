---
created: 2026-04-21
modified: 2026-05-28
entity_type: instruction
status: evergreen
title: update-entity
type: instruction
permalink: meta/agent-actions/update-entity
tags:
  - system/action
  - meta/instruction
  - type/update
  - source/largo
  - machine/largo
---

# Action: Update Entity

## Directive
Modify existing entity observations, relations, or metadata while preserving structure.

## Observations
- [operation] append: add content to end of section or file
- [operation] prepend: add content to beginning of section
- [operation] replace_section: replace entire section content
- [operation] find_replace: targeted text substitution
- [constraint] Always read current state before modification
- [constraint] Preserve existing observations when adding new ones
- [validation] Verify edit success via read_note after modification

## Execution Pattern
```
1. read_note(identifier="{entity}") → capture current state
2. edit_note(
     identifier="{entity}",
     operation="{append|prepend|replace_section|find_replace}",
     section="{Observations|Relations}",  # For section operations
     content="{new_content}",
     find_text="{old_text}",              # For find_replace only
     expected_replacements=1              # For find_replace validation
   )
3. read_note(identifier="{entity}") → verify changes
```

## Operation Selection
| Goal | Operation | Section Param |
|------|-----------|---------------|
| Add observation | append | "Observations" |
| Add relation | append | "Relations" |
| Update frontmatter | replace_section | Frontmatter field |
| Fix typo | find_replace | N/A |
| Rewrite section | replace_section | Section name |

## Content Formats
```markdown
# Adding observation
- [{category}] {new_fact} #{tag}

# Adding relation  
RELATION_LINE: {relation_type} [[{Target Entity}]]
```

## Relations
- part_of [[meta/agent-actions/index]]
- related_to [[meta/agent-actions/create-entity]]
