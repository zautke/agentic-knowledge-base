---
title: manage-taxonomy
type: instruction
permalink: meta/agent-actions/manage-taxonomy
tags:
- system/action
- meta/instruction
- type/organize
---

# Action: Manage Taxonomy

## Directive
Maintain consistent folder hierarchy, tag namespaces, and entity organization.

## Observations
- [hierarchy] Maximum 3 folder levels for optimal sync and navigation
- [hierarchy] Top-level folders map to knowledge domains: meta/, projects/, concepts/, decisions/, reference/
- [tags] Namespace format: {category}/{value} - type/, domain/, status/, project/
- [tags] Limit active tags to 50-100 with clear definitions
- [naming] Concepts: Singular noun phrase (e.g., "API Authentication")
- [naming] Projects: Prefix pattern for glob filtering (e.g., "Project - Name")
- [naming] Decisions: Date prefix for temporal queries (e.g., "2025-01-15 Topic")
- [anti-pattern] Avoid one-off folders; use tags for cross-cutting concerns
- [anti-pattern] Avoid duplicate content; use relations for cross-references
- [requirement] Taxonomy decisions must be based on neighborhood discovery, not only local folder intuition
- [requirement] Every taxonomy pass must emit an exact structural audit to the user

## Execution Patterns

### Audit Structure
```
1. list_directory(dir_name="/", depth=3) → map hierarchy
2. search_notes(query="", page_size=50) → inventory entities
3. search_notes(tags=["domain/{subject}"]) → tag neighborhood
4. search_notes(query="{subject_keywords}", search_type="hybrid") → semantic neighborhood
5. schema_diff(note_type="{dominant_type}") when note-class drift is suspected
6. Identify: orphans, deep nesting, naming inconsistencies, duplicate domains, weakly linked clusters, relation vocabulary drift, note-type drift
```

### Reorganize Entity
```
1. read_note(identifier="{entity}") → capture state
2. move_note(
     identifier="{entity}",
     destination_path="{new_folder}/{filename}.md"
   )
3. Update relations in connected entities if needed
```

### Tag Normalization
```
1. search_notes(query="tag:{old_tag}") → find affected entities
2. For each entity:
   edit_note(operation="find_replace", find_text="{old_tag}", content="{new_tag}")
```

### Deterministic Taxonomy Decision Rules
1. Check for an existing global note before creating or moving a local note into a new domain bucket.
2. Prefer existing namespace tags over inventing new tags.
3. If a note spans multiple domains, create bridge links before creating a new folder branch.
4. If the note is hard to rediscover lexically, ensure an index/hub note also points to it.
5. Record candidate placements considered and why they were rejected.
6. Reindex after broad renames, moves, or large curation batches so semantic retrieval remains trustworthy.

## Required Oversight Output
- exact notes moved, renamed, or retagged
- exact old tags -> new tags mapping
- exact relations or links added to preserve discoverability
- exact searches/context traversals used to decide placement
- exact schema drift reviewed or ignored
- unresolved taxonomy ambiguities and likely gaps

## Standard Folder Structure
```
{project}/
├── meta/           # System documentation, agent instructions
├── concepts/       # Reusable knowledge entities
├── decisions/      # Dated architectural decisions
├── projects/       # Active work contexts
├── reference/      # External resources, documentation
└── archive/        # Completed or deprecated entities
```

## Relations
- part_of [[meta/agent-actions/index]]
- implements [[Taxonomy Best Practices]]
- related_to [[Semantic Relationship Discovery and Linking Protocol]]
- related_to [[Schema Governance and Taxonomy Normalization Protocol]]
