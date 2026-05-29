---
created: 2026-04-21
modified: 2026-05-28
entity_type: meta_prompt
status: evergreen
title: extend-knowledge-graph
type: meta_prompt
permalink: meta/prompts/extend-knowledge-graph
tags:
- system/meta-prompt
- meta/extend
- type/instruction
- source/largo
- machine/largo
---

# Meta-Prompt: Extend Knowledge Graph

## Directive
Add new entities to the knowledge graph with proper taxonomy alignment and relation connectivity.

## Observations
- [precondition] Read meta/README first to load system context
- [precondition] Read meta/taxonomy/index to load naming/structure conventions
- [pattern] Every new entity MUST link to at least one existing entity
- [pattern] Use namespaced tags from established vocabulary
- [validation] Verify entity integrates into graph via build_context after creation
- [requirement] Discovery must be deterministic and auditable, not intuitive-only
- [requirement] When available, hybrid/vector search candidates must be reviewed before final taxonomy placement

## Execution Sequence

### Phase 1: Load Context
```
1. build_context(url="memory://meta/readme", depth=1) → system knowledge
2. read_note(identifier="meta/taxonomy/index") → naming conventions
3. search_notes(query="{topic_keywords}") → lexical candidate entities
4. search_notes(query="{topic_keywords}", search_type="hybrid") → semantic candidate entities
5. search_notes(tags=["domain/{subject}"]) → tag-neighborhood entities
6. build_context(url="memory://{best_existing_hub}", depth=1) → nearby graph neighborhood
7. if note class or metadata conventions are changing, review schema_diff for the affected note type
```

### Phase 2: Plan Entity
```
Determine:
- entity_type: from taxonomy/index conventions
- folder: from 3-level hierarchy constraint
- title: from naming pattern for entity_type
- tags: from established namespaces
- relations: minimum 1 to existing entity, preferably 1 hub/index plus 1 semantically adjacent note
- candidate notes considered: linked, rejected, ambiguous
- whether schema or taxonomy normalization is also required
```

### Phase 3: Create Entity
```
write_note(
  folder="{determined_folder}",
  title="{formatted_title}",
  entity_type="{type}",
  content="""
# {Title}

## Observations
- [{category}] {fact_1}
- [{category}] {fact_2}

## Relations
RELATION_LINE: {relation_type} [[{Existing Entity}]]
""",
  tags=["{namespace/tag}"]
)
```

### Phase 4: Verify Integration
```
1. build_context(url="memory://{new_permalink}", depth=1) → verify edges
2. build_context(url="memory://{linked_entity}", depth=1) → verify bidirectional if needed
3. confirm no unresolved links remain silently hidden
4. prepare exact observability report for user closeout
```

## Required User Oversight Output
- exact note path/permalink created
- exact tags added
- exact relation lines and wiki-links added
- exact discovery queries and context traversals used
- exact schema/taxonomy checks used when they influenced placement
- unresolved or ambiguous neighbors that may still need linking

## Anti-Patterns to Avoid
- Creating orphan entities (no relations)
- Using tags outside established namespaces
- Nesting folders beyond 3 levels
- Duplicate content instead of relation links
- Generic category annotations (use specific: method, decision, principle, constraint)

## Relations
- part_of [[meta/agent-actions/index]]
- uses [[meta/agent-actions/create-entity]]
- uses [[meta/taxonomy/index]]
- related_to [[Schema Governance and Taxonomy Normalization Protocol]]
