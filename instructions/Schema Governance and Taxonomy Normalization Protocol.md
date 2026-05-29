---
title: Schema Governance and Taxonomy Normalization Protocol
created: 2026-03-13
modified: 2026-03-13
type: instruction
entity_type: instruction
status: evergreen
tags:
- project/agent-kb
- domain/schema
- domain/taxonomy
- domain/curation
- op/hygiene
- status/evergreen
- source/largo
- machine/largo
permalink: agent-kb/instructions/schema-governance-and-taxonomy-normalization-protocol
---

# Schema Governance and Taxonomy Normalization Protocol

## Purpose
Deterministic procedure for reducing note-type drift, relation-vocabulary drift,
and taxonomy inconsistency using the modern Basic Memory schema tools.

## When to Use
- new note types are being introduced
- dominant note types have no stored schemas
- taxonomy passes reveal naming or metadata drift
- relation lines are diverging across similar notes

## Procedure
1. Run `schema-infer` on the dominant affected note types.
2. Run `schema-diff` to measure drift against any existing contract.
3. Inventory note-type and relation-vocabulary outliers.
4. Decide what should become:
   - canonical `type`
   - canonical `entity_type`
   - canonical relation vocabulary
   - tags instead of note-type proliferation
   - semantically distinct relation pairs that must not be collapsed, such as `part_of` vs `child_of`
5. Normalize notes incrementally.
6. Run `schema-validate`.
7. Reindex after broad metadata or taxonomy changes.

## Current Canonical Schema Notes
- [[Instruction Schema]]
- [[Reference Schema]]
- [[Index Schema]]

## Required Audit Output
- exact note types reviewed
- exact drift or absence-of-schema findings
- exact normalization rules applied
- exact notes retagged, retyped, or relinked
- exact remaining outliers and why they remain

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-03-13 | Initial schema governance protocol created. | Schema tooling exists but KB operating protocols needed a canonical way to use it for taxonomy improvement. | User request to execute the earlier KB capability audit recommendations. |

## Relations

- related_to [[Agentic Knowledge Base System Overview]]
- related_to [[Basic Memory Capability Audit and KB Optimization Recommendations]]
- related_to [[Curator Session Primer and Runbook]]
- related_to [[Semantic Relationship Discovery and Linking Protocol]]
- related_to [[Instruction Schema]]
- related_to [[Reference Schema]]
- related_to [[Index Schema]]
