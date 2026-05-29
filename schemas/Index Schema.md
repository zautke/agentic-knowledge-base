---
title: Index Schema
created: 2026-03-13
modified: 2026-05-28
type: schema
entity_type: reference
status: evergreen
entity: index
version: 1
schema:
  related_to?(array): Reference
settings:
  validation: warn
tags:
- project/agent-kb
- domain/schema
- domain/indexing
- status/evergreen
permalink: agent-kb/schemas/index-schema
---

# Index Schema

Schema contract for notes with `type: index`.

## Purpose
- establish a minimal executable contract for hub and navigation notes
- keep index notes structurally simple while exposing relation drift
- support later expansion if the KB adopts stronger hub semantics

## Scope
- target note type: `index`
- validation mode: `warn`
- current focus: the dominant `related_to` edge used in existing hub notes

## Related
- [[Schema Governance and Taxonomy Normalization Protocol]]
- [[Curator Session Primer and Runbook]]
- [[Basic Memory Capability Audit and KB Optimization Recommendations]]

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-05-28 | Added the mandatory append-only Evolution Log and a typed Relations block; bumped `modified`. The existing `## Related` link list is retained verbatim (accumulate, not replace). | Schema notes are living governance contracts but lacked the Evolution Log required by Metaprompt Rule 5 and carried only bare wiki-links rather than typed relations. | Governance-doc curation sweep over `schemas/`. |

## Relations

- governed_by [[Schema Governance and Taxonomy Normalization Protocol]]
- used_by [[Curator Session Primer and Runbook]]
- related_to [[Basic Memory Capability Audit and KB Optimization Recommendations]]
- indexed_by [[Index – Agent KB]]
