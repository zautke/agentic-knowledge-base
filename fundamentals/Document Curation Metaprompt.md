---
title: Document Curation Metaprompt
created: 2026-01-01
modified: 2026-02-16
type: fundamental
entity_type: fundamental
status: evergreen
permalink: fundamentals/document-curation-metaprompt
tags:
- project/agent-kb
- domain/documentation
- domain/curation
- system/core
- status/evergreen
---

# Document Curation Metaprompt

The universal self-maintenance protocol for any living document in the knowledge base. Every structured document, note, protocol, runbook, manual, or specification that is intended to evolve through use MUST include an adapted version of this metaprompt.

This is a **fundamental** -- it applies to ALL knowledge base operations, not just document creation.

## The 8 Governing Principles

These rules are derived from the ACE (Agentic Context Engineering) framework and validated through production use on PRDs, architecture docs, and agent manuals.

### Rule 1: ACCUMULATE, NEVER SUMMARIZE AWAY

Documents grow by accretion. New learnings, patterns, and anti-patterns are ADDED to their respective sections. Do not rewrite existing entries to be "more concise" -- this causes **context collapse**, where iterative editing erodes domain-specific detail that future agents need.

**Brevity bias** (dropping specific examples for abstract principles) degrades a document's utility over time. The cost of extra tokens is lower than the cost of lost institutional knowledge.

### Rule 2: STRUCTURED INCREMENTAL UPDATES ONLY

Every modification must target a specific, named section. Additions go into the appropriate registry (Anti-Pattern Registry, Technique Library, Decision Record, or Evolution Log). Do not perform whole-document rewrites unless the user explicitly requests a restructuring pass.

### Rule 3: EVIDENCE-LINKED ENTRIES

Every new entry in any registry must include:
- **Date** of discovery
- **Context** (which project, which document, what went wrong or right)
- **Concrete example** (not abstract principle alone)
- **Source** (research paper, production incident, user feedback, etc.)

Entries without provenance are indistinguishable from hallucinated advice.

### Rule 4: SELF-REFERENTIAL VERIFICATION

Before adding a new technique or anti-pattern, check whether it contradicts or duplicates an existing entry. If it contradicts, do not silently overwrite -- add the new entry with an explicit note about the tension and under what conditions each applies.

### Rule 5: EVOLUTION LOG IS MANDATORY

Every modification to a living document must be recorded in its Evolution Log with: date, what changed, why, and what triggered the change. This log is **append-only**. It is **never pruned**.

### Rule 6: TRIGGER CONDITIONS FOR UPDATE (MUST BE ADAPTED)

This is the only rule that MUST be customized per document. Generic triggers produce generic (useless) curation. Define specific events that demand an update.

**Template triggers (adapt these):**
- a. The document is used and the process reveals a gap or new technique
- b. New research on the document's domain is encountered
- c. A review identifies sections that were weak or strong
- d. A new domain not previously covered is encountered
- e. The user explicitly requests an update

### Rule 7: PROTECTED SECTIONS

- Do not modify the metaprompt section without user consent
- Do not remove entries from registries (mark as deprecated with date and reason instead)
- Do not change section numbering (breaks cross-references)
- Do not add emoji unless the user explicitly requests it

### Rule 8: QUALITY SIGNAL TRACKING

### Rule 9: CURATOR AGENT GATING — NON-NEGOTIABLE

No knowledge enters the KB without passing through the `curator` agent. The author of a note is never its approver. This separation is structural, not procedural — enforced by tool access, not convention.

**Submitting agents:** prepare, package, submit. Do not write.
**Curator agent:** score, judge, write (or reject with specific feedback).

The minimum passing score is **91/100**. Submissions scoring ≤90 are rejected and must be resubmitted after fixing the specific issues identified.

This rule cannot be adapted, bypassed, or soft-pedaled for convenience. It exists because the test proved the system fails without it — a submission scoring 47/100 entered the KB because the author was also the approver.

After each document is used, the agent should assess and record:
- Which sections were most useful
- Which sections were ignored or inapplicable
- Which sections had insufficient guidance (agent had to improvise)
- Total creation/usage time and token usage (if measurable)
- User satisfaction signal (explicit feedback or inferred from revision requests)

## Structural Observability Requirement

When a curation pass changes semantic structure, the curator must produce a user-facing audit trail that is exact enough for oversight.

Minimum required contents:
- exact note identifiers or paths touched
- exact tags added, removed, normalized, or retained deliberately
- exact relation blocks added, removed, or changed
- exact wiki-links added, removed, or left unresolved
- the discovery process used to justify those choices, including searches, neighboring notes reviewed, and context traversals performed
- explicit statement of what may still be missing, ambiguous, or weakly justified

This requirement exists to prevent silent taxonomy drift and to expose missed-neighbor risk to the user.

## Applying to Knowledge Base Notes

These 8 rules do not apply only to standalone documents. They apply to **any note in the knowledge base that is intended to persist and evolve**:

| Note Type | How to Apply |
|-----------|-------------|
| **Fundamentals** | Rules 1, 2, 4, 7 are critical. Never summarize away root knowledge. |
| **Instructions** | Rules 2, 3, 5 are critical. Every update to a procedure must be logged. |
| **References** | Rules 1, 3, 4 are critical. Accumulate, cite sources, check for contradictions. |
| **Indices** | Rules 2, 7 are critical. Incremental updates only; do not restructure without consent. |
| **Protocols** | All 8 rules apply. Protocols are living documents by definition. |
| **Project notes** | Rules 1, 2, 3, 5 are critical. Project knowledge must accumulate with evidence. |

## The Metaprompt Template

When creating a new living document, include this adapted block at the top:

```
ROLE: You are a document curator for [DOCUMENT_NAME] -- an evolving
[document type] that [one-sentence purpose]. This document is a living
artifact that improves through structured use.

GOVERNING PRINCIPLES:
1. ACCUMULATE, NEVER SUMMARIZE AWAY [verbatim]
2. STRUCTURED INCREMENTAL UPDATES ONLY [verbatim]
3. EVIDENCE-LINKED ENTRIES [verbatim]
4. SELF-REFERENTIAL VERIFICATION [verbatim]
5. EVOLUTION LOG IS MANDATORY [verbatim]
6. TRIGGER CONDITIONS FOR UPDATE [ADAPT per document]
7. WHAT NOT TO CHANGE [verbatim + extend if needed]
8. QUALITY SIGNAL TRACKING [verbatim, reference actual sections]
```

Rules 1-5 are verbatim across ALL documents. Rule 6 MUST be adapted. Rules 7-8 may be extended.

## Observations

- [fundamental] The 8 rules are derived from ACE framework (UC Berkeley/Stanford, Oct 2025) and validated through production use
- [fundamental] Context collapse occurs when iterative editing erodes domain-specific detail; the antidote is accumulate-never-summarize
- [fundamental] Brevity bias (dropping examples for abstract principles) is the most common failure mode in agent-maintained documents
- [constraint] Rule 6 (trigger conditions) MUST be adapted per document; generic triggers produce useless curation
- [constraint] Rules 1-5 are never modified per document; they are universal ACE principles
- [best-practice] Evidence-linked entries prevent hallucinated advice from entering registries
- [best-practice] Append-only evolution logs create provenance chains that future agents can audit
- [anti-pattern] Whole-document rewrites cause context collapse and lose institutional knowledge
- [anti-pattern] Entries without date/context/example/source are indistinguishable from hallucination
- [source] ACE: Agentic Context Engineering (UC Berkeley/Stanford, Oct 2025)
- [source] MetaReflection (Microsoft Research, May 2024): 4-16.82% performance boost from distilled experiential learnings
- [source] PRD_DOCUMENT_CREATION.md: first production application of these principles

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-02-16 | Metadata and relation hygiene pass | Normalized frontmatter to KB contract and standardized relation syntax for graph consistency. | Ongoing KB hygiene sweep. |
| 2026-02-16 | Relation keyword normalization | Converted relation lines to canonical `verb [Target]` style to avoid parser variance in relation types. | Follow-up graph hygiene pass. |

## Relations

- part_of [[Fundamental – Agent KB]]
- implements [[Agentic Knowledge Base System Overview]]
- used_by [[Agentic Living Document Playbook]]
- used_by [[PRD Document Creation Playbook]]
- related_to [[Action Taxonomy – Index]]
- related_to [[Instruction – Update Node]]
- related_to [[Instruction – Hygiene or Compression]]
