---
title: index
type: index
permalink: meta/taxonomy/index
tags:
- system/core
- meta/taxonomy
- type/index
---

# Taxonomy Index

## Purpose
Defines the canonical structure, naming conventions, and organizational patterns for this knowledge base.

## Observations
- [structure] Three-level maximum folder depth enforced for all entities
- [structure] Top-level domains: meta/, concepts/, decisions/, projects/, reference/, archive/
- [convention] Entity types: concept, decision, project, reference, meeting, person, instruction, index
- [convention] Status lifecycle: draft → active → review → evergreen → archived
- [tags] Namespaces: type/, domain/, status/, project/, system/
- [relation] Canonical relation types: implements, uses, related_to, part_of, child_of, successor_of, applies_to
- [relation] `part_of` means compositional containment; `child_of` means scoped membership or affiliation and must remain distinct

## Folder Purposes
| Folder | Entity Types | Naming Pattern |
|--------|-------------|----------------|
| meta/ | instruction, index, knowledge-base | descriptive lowercase |
| concepts/ | concept | Singular Noun Phrase |
| decisions/ | decision | YYYY-MM-DD Topic |
| projects/ | project | Project - Name |
| reference/ | reference | Source - Title |
| archive/ | any | Original name preserved |

## Tag Namespace Reference
| Namespace | Purpose | Examples |
|-----------|---------|----------|
| type/ | Entity classification | type/concept, type/decision |
| domain/ | Subject area | domain/architecture, domain/ml |
| status/ | Workflow state | status/active, status/archived |
| project/ | Project association | project/agent-kb |
| system/ | Infrastructure | system/core, system/action |

## Relations
- part_of [[meta/README]]
- implements [[Taxonomy Best Practices]]
