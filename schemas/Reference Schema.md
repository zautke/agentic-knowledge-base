---
title: Reference Schema
created: 2026-03-13
modified: 2026-03-13
type: schema
entity_type: reference
status: evergreen
entity: reference
version: 1
schema:
  related_to?(array): Reference
  child_of?: Reference
  uses?: Reference
settings:
  validation: warn
tags:
- project/agent-kb
- domain/schema
- domain/reference
- status/evergreen
permalink: agent-kb/schemas/reference-schema
---

# Reference Schema

Schema contract for notes with `type: reference`.

## Purpose
- establish an executable baseline for the dominant conceptual note class
- surface relation-vocabulary drift around `related_to` and competing aliases
- support incremental normalization rather than immediate hard enforcement

## Scope
- target note type: `reference`
- validation mode: `warn`
- current focus: the most common explicit relation shapes in live notes

## Related
- [[Schema Governance and Taxonomy Normalization Protocol]]
- [[Curator Session Primer and Runbook]]
- [[Basic Memory Capability Audit and KB Optimization Recommendations]]
