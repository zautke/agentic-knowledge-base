---
title: Curator Session Primer and Runbook
created: 2026-03-13
modified: 2026-03-13
type: instruction
entity_type: instruction
status: evergreen
tags:
- project/agent-kb
- domain/curation
- domain/semantic-search
- domain/schema
- op/session-start
- role/curator
- status/evergreen
permalink: agent-kb/instructions/curator-session-primer-and-runbook
---

# Curator Session Primer and Runbook

## Purpose
Canonical runbook for sessions where the main output is better knowledge-base
quality, discoverability, and agent operating leverage.

## Trigger Conditions
- taxonomy or linking sweeps
- schema governance work
- unresolved-link cleanup
- orphan and weak-cluster review
- semantic-search maintenance
- protocol, primer, or runbook hardening

## Required Context Injection
1. Read [[Agentic Knowledge Base System Overview]].
2. Read [[Document Curation Metaprompt]].
3. Read [[Curator Handover Knowledge]].
4. Read [[Semantic Relationship Discovery and Linking Protocol]].
5. Check `project info` for semantic-search and maintenance state.
6. Check `schema-diff` / `schema-validate` if note classes or taxonomy may change.

## Deterministic Discovery Sequence
1. lexical search
2. hybrid/vector search when embeddings are current
3. tag-neighborhood search
4. `build_context` on hubs and nearest candidates
5. review unresolved relations or isolated notes when relevant

## Required Structural Outputs
- updated notes, tags, and relation lines
- repaired hubs or bridge links
- schema or note-type normalization actions
- reindexing when needed
- exact user-facing structural audit

## Oversight Requirements
Every closeout must include:
- exact notes changed
- exact tags changed
- exact relation lines and links changed
- exact discovery method used
- exact unresolved or ambiguous candidates
- exact maintenance actions and measured duration when timing was captured

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-03-13 | Initial Curator session runbook created. | KB-only sessions needed a single high-context procedural entry point for modern semantic and schema-aware curation. | User request to formalize Curator primer and runbook. |

## Relations

- related_to [[Curator Handover Knowledge]]
- related_to [[Basic Memory Semantic Reindex Protocol]]
- related_to [[Semantic Relationship Discovery and Linking Protocol]]
- related_to [[Schema Governance and Taxonomy Normalization Protocol]]
