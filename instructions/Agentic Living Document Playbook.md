---
title: Agentic Living Document Playbook
created: 2026-01-01
modified: 2026-02-16
type: instruction
entity_type: instruction
status: evergreen
permalink: instructions/agentic-living-document-playbook
tags:
- project/agent-kb
- domain/documentation
- domain/methodology
- status/evergreen
- op/create-node
---

# Agentic Living Document Playbook

A document-type agnostic procedural playbook for creating and maintaining rigorous, grounded living documents of any type. This is the parent methodology from which type-specific playbooks (PRDs, ADRs, runbooks, etc.) are derived.

**Full document location:** `/Volumes/FLOUNDER/dev/fastapi/AGENTIC_LIVING_DOCUMENT_PLAYBOOK.md`
**Comprehensive version (agentic-kb):** `00_Meta/Agentic Living Document Playbook`

## Observations

- [methodology] Living documents have four properties: Structured (defined section schema), Grounded (traceable to evidence), Self-improving (mechanisms for agent curation), Versioned (append-only evolution log)
- [methodology] Document creation follows five sequential phases: Research (3+ disparate sources) -> Reconnaissance (audit actual subject matter) -> Structure (section selection) -> Composition (write each section) -> Verification (quality gates)
- [architecture] Every living document must contain 5 universal sections: Agent Curation Metaprompt, Purpose and Scope, Content Sections, Anti-Pattern Registry, Evolution Log
- [technique] 13 named techniques: Dual-Value Targets (T-1), Degradation Matrix (T-2), Three-Layer Evaluation (T-3), Non-Goals Test (T-4), Gap Table (T-5), Phased Tables (T-6), Risk-Level Grid (T-7), Scalability Escape Hatches (T-8), Consequence-Driven Gates (T-9), ASCII Flow Diagrams (T-10), Decision Matrix (T-11), Numbered Steps with Verification (T-12), Timestamped Event Log (T-13)
- [anti-pattern] 12 documented anti-patterns including: writing before examining subject matter, whole-document rewrites, generic metaprompts, duplicating content, skipping non-goals
- [adaptation] Document type guide covers 7 types: PRD, ADR, Runbook, Protocol, Technical Specification, Instructional Manual, Postmortem
- [constraint] Specialized playbooks inherit from this parent but govern for their specific type when conflicts arise

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-02-16 | Metadata and relation hygiene pass | Normalized frontmatter to KB contract and standardized relation syntax for graph consistency. | Ongoing KB hygiene sweep. |
| 2026-02-16 | Relation keyword normalization | Converted relation lines to canonical `verb [Target]` style to avoid parser variance in relation types. | Follow-up graph hygiene pass. |

## Relations

- parent_of [[PRD Document Creation Playbook]]
- implements [[Document Curation Metaprompt]]
- related_to [[Agentic Knowledge Base System Overview]]
- related_to [[Action Taxonomy – Index]]