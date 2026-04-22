---
title: link-entities
type: instruction
permalink: meta/agent-actions/link-entities
tags:
- system/action
- meta/instruction
- type/link
---

# Action: Link Entities

## Directive
Establish typed directed edges between existing graph nodes.

## Observations
- [constraint] Canonical relation vocabulary must stay semantically distinct; do not collapse `child_of` into `part_of`
- [constraint] Core/high-frequency relation set: implements, uses, related_to, part_of, child_of, successor_of, applies_to
- [constraint] Target entity must exist or relation remains unresolved until creation
- [syntax] Relation format: `- {relation_type} [[{Target Title}]]`
- [syntax] Optional context suffix: `- {relation_type} [[{Target}]] (contextual note)`
- [bidirectional] For symmetric relationships, create relations in both entities
- [validation] Use build_context after linking to verify edge creation
- [requirement] Linking must begin with a deterministic candidate-discovery pass
- [requirement] The user must receive an exact list of links considered, created, and left unresolved

## Execution Pattern
```
1. read_note(identifier="{source_entity}") → get current content
2. search_notes(query="{target_keywords}") → lexical candidates
3. search_notes(query="{target_keywords}", search_type="hybrid") → semantic candidates when enabled
4. search_notes(tags=["domain/{subject}"]) → tag-neighborhood candidates
5. build_context(url="memory://{source_permalink}", depth=1) → inspect current neighborhood
6. search_notes(query="{target_entity}") → verify target exists
7. if similarity or ambiguity matters, record candidate score signals from hybrid/vector search
8. choose relation_type and whether bidirectional linking is justified
9. if relation vocabulary drift is present, check schema/taxonomy guidance first
8. edit_note(
     identifier="{source_entity}",
     operation="append",
     section="Relations",
     content="- {relation_type} [[{Target Entity}]]"
   )
10. build_context(url="memory://{source_permalink}", depth=1) → verify edge
11. prepare exact observability report
```

## Relation Type Semantics
| Type | Direction | Use Case |
|------|-----------|----------|
| implements | Source → Spec | Realization of design/pattern |
| uses | Source → Dependency | Functional dependency |
| related_to | Bidirectional | Associative connection |
| part_of | Child → Parent | Hierarchical composition |
| child_of | Child → Parent Entity | Scoped membership or affiliation without claiming compositional containment |
| applies_to | Protocol → Target Domain | Instruction, rule, or method applicability |
| successor_of | New → Old | Temporal/version evolution |

## Required Oversight Output
- exact source note and target note
- exact relation line added
- exact candidate notes searched before choosing the target
- exact score or ranking signals consulted when semantic search informed the decision
- exact reason this relation type was chosen instead of alternatives
- unresolved candidate links and rejected candidates

## Relations
- part_of [[meta/agent-actions/index]]
- related_to [[meta/agent-actions/create-entity]]
- implements [[Semantic Relationship Discovery and Linking Protocol]]
- related_to [[Schema Governance and Taxonomy Normalization Protocol]]
