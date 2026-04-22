---
title: PRD Document Creation Playbook
created: 2026-01-01
modified: 2026-02-16
type: instruction
entity_type: instruction
status: evergreen
permalink: instructions/prd-document-creation-playbook
tags:
- project/agent-kb
- domain/documentation
- domain/product-requirements
- status/evergreen
- op/create-node
---

# PRD Document Creation Playbook

An agentic instructional manual for creating rigorous, grounded Product Requirements Documents. This is a domain-specific specialization of the [[Agentic Living Document Playbook]] applied to PRDs.

**Full document location:** `/Volumes/FLOUNDER/dev/fastapi/PRD_DOCUMENT_CREATION.md` (788 lines)
**Comprehensive version (agentic-kb):** `manuals/PRD Document Creation Playbook`

**Proven on:** FastAPI ML Platform PRD (599 lines, 10 sections, AI/ML domain)

## Observations

- [methodology] PRD creation follows five sequential phases: Research (3+ disparate sources) -> Reconnaissance (codebase audit) -> Structure (section selection) -> Composition (section-by-section writing) -> Verification (quality gates)
- [technique] Dual-Value Performance Targets (T-1): every metric needs Minimum Acceptable + Target + Conditions
- [technique] Per-Service Degradation Matrix (T-2): for each failing service, specify Impact + Fallback + User-Visible Effect
- [technique] Three-Layer Evaluation (T-3): pre-launch tests + ongoing monitoring + evaluation datasets
- [technique] Non-Goals Test (T-4): each non-goal must reject a feature request without further discussion
- [technique] Gap Table (T-5): map components to Exists/Not Wired/Protocol Only/Missing
- [technique] Phased Endpoint Tables (T-6): organize API endpoints by implementation phase
- [technique] Risk-Level Grid (T-7): classify AI outputs as Low/Medium/High risk
- [technique] Scalability with Escape Hatches (T-8): v1.0 Target + Scaling Path
- [technique] Consequence-Driven Phase Gates (T-9): each gate includes Blocker If Missed
- [technique] ASCII Data Pipeline Diagrams (T-10): renders everywhere unlike Mermaid
- [anti-pattern] Writing PRD before reading codebase produces fantasy documents
- [anti-pattern] Single-value performance targets provide no broken vs. aspirational distinction
- [anti-pattern] "Gracefully degrade" without specifics gives engineering zero guidance
- [anti-pattern] Treating all AI outputs as hallucination-prone causes over-engineering
- [anti-pattern] Omitting research sources removes provenance for future updates
- [constraint] PRDs require 4 inputs: codebase access, architecture decisions, roadmap, user input
- [constraint] AI/ML PRDs additionally require: model inventory, data store topology, evaluation criteria

## Self-Maintenance Protocol

Includes Agent Curation Metaprompt with 8 binding rules from the ACE framework. See [[Document Curation Metaprompt]] for the universal template.

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-02-16 | Metadata and relation hygiene pass | Normalized frontmatter to KB contract and standardized relation syntax for graph consistency. | Ongoing KB hygiene sweep. |
| 2026-02-16 | Relation keyword normalization | Converted relation lines to canonical `verb [Target]` style to avoid parser variance in relation types. | Follow-up graph hygiene pass. |

## Relations

- specializes [[Agentic Living Document Playbook]]
- implements [[Document Curation Metaprompt]]
- related_to [[Agentic Knowledge Base System Overview]]
- related_to [[Action Taxonomy – Index]]