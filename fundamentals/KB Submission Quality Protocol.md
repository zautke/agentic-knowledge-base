---
title: KB Submission Quality Protocol
type: instruction
permalink: agent-kb/fundamentals/kb-submission-quality-protocol
created: 2026-04-19
modified: 2026-04-19
entity_type: instruction
status: evergreen
tags:
- project/agent-kb
- domain/curation
- role/curator
- op/protocol
- op/quality-gate
---

# KB Submission Quality Protocol

The Curator is a quality gate. Every addition or update to the knowledge base is scored before acceptance. Submissions scoring ≤90/100 are **rejected** with specific gate-level feedback and must be resubmitted.

This protocol applies to all submissions: new notes, updates to existing notes, bulk ingestion runs, and automated pipeline outputs.

---

## Authority Constraint — Non-Negotiable

Only the `curator` agent (`/agents/curator.md`) has authority to write to the knowledge base.

**No other agent, skill, or process may call `write_note` or `edit_note`.** This is enforced at the tool level — no other agent has these tools in its allowlist.

### Submitting Agent Workflow

If you are not the `curator` agent, your KB submission workflow is:

1. Prepare a complete submission package (see Submission Interface in the curator agent)
2. Spawn the curator: `Agent(subagent_type="curator", prompt=<submission package>)`
3. Await the curator's decision
4. If rejected: fix exactly the identified issues and resubmit
5. Task is NOT complete until the curator issues an approval

### Exemption: Structural Scaffolding

`kb:project-init` is exempt from curator scoring. It creates blank placeholder notes for project structure — not curated knowledge. Blank scaffolding cannot be scored. This exemption is documented and limited strictly to `project-init`.

Everything else — new notes, note updates, hub index changes, metadata edits — goes through the curator.

## The Scoring Rubric

Total possible: **100 points**. Minimum passing score: **91**.

| Gate | Weight | What Is Checked |
|------|--------|-----------------|
| Gate 0 — Authorization | 10 | Non-obvious? Saves ≥1 future-agent search? Stable >2 months? Not code-derivable? |
| Gate 1 — Classification | 10 | Correct directory per Bridge Protocol? Correct `type`? Global truths not buried in project folders? |
| Gate 2 — Deduplication | 10 | All 4 dedup checks completed before `write_note`? No duplicate created? |
| Gate 3 — Composition & Enrichment | 20 | All 7 frontmatter fields (10 pts)? KB fork enriched beyond source — not a mirror (10 pts)? |
| Gate 4 — Linking | 25 | Hub or index note updated with inbound `[[link]]` (15 pts)? ≥1 explicit relation in note body (10 pts)? |
| Gate 5 — Verification | 10 | No internal contradictions (5 pts)? Unresolved links annotated with reason (5 pts)? |
| Gate 6 — Oversight Report | 15 | Report produced (5 pts)? All required fields present (5 pts)? Accurate and specific (5 pts)? |

Gate 4 and Gate 6 together represent **40% of the score**. An orphan note with no oversight report cannot pass.

---

## Scoring Sub-criteria Detail

### Gate 0 — Authorization (10 pts)

Hard reject if any one condition is true (scores 0):
- Trivial implementation detail visible in code or docstrings
- Ephemeral state or in-progress work
- Content recoverable in a single file read

### Gate 1 — Classification (10 pts)

| Score | Condition |
|-------|-----------|
| 10/10 | Correct type, correct directory, Bridge Protocol applied correctly |
| 5/10 | Correct type, wrong directory (e.g., global truth buried in project folder) |
| 0/10 | Wrong type AND wrong directory |

**Bridge Protocol reminder:**
- Global truth → `reference/patterns/`
- Project-specific → `projects/<slug>/knowledge/`
- Hybrid → local note + `[[link]]` to global

### Gate 2 — Deduplication (10 pts)

All 4 checks required before `write_note`:
1. `search_notes(query=<topic>)` — lexical
2. `search_notes(query=<topic>, search_type="hybrid")` — semantic
3. `search_notes(tags=["project/<slug>", "domain/<subject>"])` — tag-neighborhood
4. `build_context(url="projects/<slug>/", depth=1)` — graph expansion

| Score | Condition |
|-------|-----------|
| 10/10 | All 4 checks completed, no duplicates created |
| 5/10 | 2–3 of 4 checks completed |
| 0/10 | Dedup skipped, or note written despite a matching note existing |

### Gate 3 — Composition & Enrichment (20 pts)

**Frontmatter (10 pts):**

| Score | Condition |
|-------|-----------|
| 10/10 | All 7 fields: `title`, `created`, `modified`, `type`, `entity_type`, `status`, `tags` |
| 7/10 | 6 of 7 fields |
| 0/10 | Fewer than 5 fields |

**Enrichment (10 pts):**

The KB fork is not a reformatted copy. It must add value the filesystem doc does not have: agent-facing context, retrieval-relevant framing, relationships to adjacent concepts.

| Score | Condition |
|-------|-----------|
| 10/10 | Adds context, framing, or connections not present in source |
| 5/10 | Reformatted or lightly annotated copy |
| 0/10 | Verbatim mirror of source document |

### Gate 4 — Linking (25 pts)

**Hub update (15 pts):**

| Score | Condition |
|-------|-----------|
| 15/15 | ≥1 hub or index note updated with an inbound `[[link]]` to the new note |
| 0/15 | No hub updated (orphan node) |

**Explicit relations (10 pts):**

| Score | Condition |
|-------|-----------|
| 10/10 | ≥1 explicit relation in note body with correct verb (`Implements`, `Related to`, `Derived from`, `Extends`, `Used by`) |
| 5/10 | Relation present but target unresolved and not annotated |
| 0/10 | No relations |

### Gate 5 — Verification (10 pts)

- 5/5: No internal contradictions
- 5/5: All `[[wiki-links]]` resolve, OR unresolved links annotated with accepted reason

### Gate 6 — Oversight Report (15 pts)

Required fields in the report:
- Exact note paths created or updated
- Exact tags applied
- Exact relation statements (verb + target)
- Exact hub notes updated and what link was added
- Dedup discovery method (all 4 queries used)
- Unresolved links with reasons
- Remaining gaps

| Score | Condition |
|-------|-----------|
| 15/15 | All fields present, specific, and accurate |
| 8/15 | Report present but vague or missing 1–2 required fields |
| 0/15 | No report produced |

---

## Rejection Protocol

When a submission scores ≤90/100, the Curator:

1. **Does not merge the note** — does not update hub indexes or set `status: evergreen`
2. **Issues a rejection report** with score breakdown, specific failing items, and fix instructions
3. **Marks the note** `status: draft` if already written

### Rejection Report Template

```
SUBMISSION REJECTED — Score: [X]/100 (minimum: 91)

Gate breakdown:
  Gate 0  (Authorization):   [X]/10 — [specific finding]
  Gate 1  (Classification):  [X]/10 — [specific finding]
  Gate 2  (Deduplication):   [X]/10 — [specific finding]
  Gate 3  (Composition):     [X]/20 — [specific finding]
  Gate 4  (Linking):         [X]/25 — [specific finding]
  Gate 5  (Verification):    [X]/10 — [specific finding]
  Gate 6  (Oversight):       [X]/15 — [specific finding]

Required fixes before resubmission:
  1. [Gate N] [Specific action required]
  2. [Gate N] [Specific action required]
  ...

Resubmit for re-scoring when all fixes are applied.
```

---

## Submission Checklist (for submitting agents)

Before calling `write_note` or submitting for curator review:

- [ ] Gate 0: Content is non-obvious, stable, and not trivially code-derivable
- [ ] Gate 1: Directory and `type` match Bridge Protocol classification
- [ ] Gate 2: All 4 dedup checks run; no existing note ignored
- [ ] Gate 3: All 7 frontmatter fields present; KB fork adds enrichment beyond source
- [ ] Gate 4: ≥1 hub note updated with inbound `[[link]]`; ≥1 relation in note body
- [ ] Gate 5: No contradictions; unresolved links annotated
- [ ] Gate 6: Oversight report produced with all required fields

A submission that passes this checklist should score ≥91.

---

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-04-19 | Initial protocol created | Codify quality gate from evaluation of two-fork-documentation skill run | Test submission scored 47/100; Gates 4 and 6 failed due to absent protocol |

## Relations

- implements [[Curator Handover Knowledge]]
- implements [[Document Curation Metaprompt]]
- implements [[Agentic Knowledge Base System Overview]]
- used_by [[two-fork-documentation skill]]
- related_to [[Codebase KB Ingestion Protocol]]
- related_to [[Action Taxonomy – Index]]