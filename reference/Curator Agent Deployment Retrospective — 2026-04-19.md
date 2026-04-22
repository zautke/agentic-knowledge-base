---
title: Curator Agent Deployment Retrospective — 2026-04-19
type: reference
permalink: agent-kb/reference/curator-agent-deployment-retrospective-2026-04-19
created: 2026-04-19
modified: 2026-04-19
entity_type: reference
status: evergreen
tags:
- project/agent-kb
- domain/curation
- pattern/separation-of-concerns
---

# Curator Agent Deployment Retrospective — 2026-04-19

## Context
Single-session retrospective of the work that (a) identified a structural flaw in KB curation, (b) designed and deployed a dedicated `curator` Claude Code agent with exclusive KB write authority, (c) retuned the fundamental protocols to make the constraint non-negotiable, and (d) validated the result with a 10-submission acceptance test. Written immediately after the test concluded; captures intent, decisions, outcomes, and remaining gaps while memory is intact.

## Trigger — the 47/100 incident
A dry-run of the `two-fork-documentation` skill asked the acting agent to self-score its own KB submission. It returned **47/100** on the 7-gate rubric and then submitted the note anyway. Two reported gaps ("unresolved link," "missing hub inbound") were marked "accepted unresolved" with no justification. The user's reaction, verbatim:

> "the entire purpose of the curation protocol is to NEVER see shit like that. NO SHIT ENTERS THE KB."

The diagnosis was structural, not procedural: the author of a note cannot credibly be its approver. No amount of instruction tuning fixes a conflict-of-interest. The only reliable fix is **separation of concerns enforced at the tool-access layer**, not at the prompt layer.

## Design — the Curator architecture

### Core invariant
```
Only the `curator` agent holds `write_note` and `edit_note`.
Every other agent, skill, and process is read-only on the KB.
```

This is enforced by the Claude Code `allowed-tools` frontmatter on skills and the `tools` field on agent definitions — not by convention, not by prompt instruction. An agent that lacks the tool in its allowlist literally cannot invoke it.

### Flow
1. Submitting agent composes note + hub target + dedup evidence + Gate 6 oversight report (the "submission package").
2. Submitter spawns `Agent(subagent_type="curator", prompt=<package>)`.
3. Curator bootstraps via the KB priming sequence.
4. Curator re-runs its own dedup independently (never trusts submitter's evidence).
5. Curator scores on the 7-gate rubric (0–100).
6. Score ≥ 91 → curator calls `write_note` + `edit_note` on the hub → issues approval report with permalink.
7. Score ≤ 90 → curator DOES NOT write → issues rejection report with gate-by-gate breakdown and specific resubmit instructions.

### Scaffold exemption (Project-Init)
One exception was carved out explicitly: the `project-init` skill creates blank placeholder notes to establish directory structure. Blank scaffolding cannot be meaningfully scored against a 7-gate rubric that expects composition and linking. The exemption is documented in the skill itself, strictly limited to scaffold creation. All non-scaffold writes still route through the curator.

### Bootstrap exception
The first three KB writes (updates to fundamentals) and the first KB note for the two-fork-documentation skill happened BEFORE the curator agent was registered in the session's agent registry. These were done by the main agent under a "bootstrap exception" — documented self-scoring of the initial deployment writes — after which all subsequent writes flowed through the curator. The agent registry loads at session start and does not hot-reload, so this exception is only reachable on first-time deployment.

## Deliverables shipped

| Artifact | Type | Purpose |
|---|---|---|
| `/Users/luke/.agents/platform/claude/agents/curator.md` | new | The Curator agent: `model: opus`, `color: red`, sole holder of write/edit/delete KB tools |
| `~/.claude/skills/two-fork-documentation/SKILL.md` | rewrite | Strips write tools; adds 6-step submission workflow; quality bar shown up front |
| `~/.claude/skills/kb/codebase-ingestion/SKILL.md` | edit | Strips write tools; references quality protocol; curator-submit step per tier |
| `~/.claude/skills/kb/session-startup/SKILL.md` | edit | Strips write/edit/delete/move tools |
| `~/.claude/skills/kb/project-init/SKILL.md` | edit | Documents scaffold exemption |
| `fundamentals/KB Submission Quality Protocol` | new/edit | 7-gate rubric (100 pts, min 91) + rejection protocol + Authority Constraint section |
| `fundamentals/curator-handover-knowledge` | edit | Section 7 (Quality Gate Role) with tool-level enforcement detail |
| `fundamentals/document-curation-metaprompt` | edit | Rule 9: CURATOR AGENT GATING — NON-NEGOTIABLE |
| `instructions/Two-Fork Documentation — Skill Reference` | new | Bootstrap-exception KB note for the skill itself |

## The 7-gate rubric (codified)

| Gate | Weight | Checks |
|---|--:|---|
| 0 — Authorization | 10 | Non-obvious? Saves ≥1 future search? Stable? Not code-derivable? |
| 1 — Classification | 10 | Correct directory (Bridge Protocol)? Correct `type`? Scope matches placement? |
| 2 — Deduplication | 10 | All 4 dedup checks in submission? (Curator re-runs independently) |
| 3 — Composition & Enrichment | 20 | All 7 frontmatter fields (10) + genuine enrichment beyond source (10) |
| 4 — Linking | 25 | Hub targeted (15) + ≥1 explicit relation in body (10) |
| 5 — Verification | 10 | No contradictions (5) + unresolved links documented (5) |
| 6 — Oversight Report | 15 | Complete report with all required fields |
| **Total** | **100** | **Minimum 91** |

Gates 4 + 6 together account for 40% of the score. An undocumented orphan cannot pass. This is deliberate: hub integration and explicit observability are the two most commonly under-specified aspects of KB submissions, so they carry the highest combined weight.

## Acceptance test — 10 submissions

Ten dummy notes were patiently curated, submitted one at a time, scored by the curator, then deleted. The test was explicitly designed to pass — the question was not "can a mediocre submission fail" but "does the curator award full marks when the submission deserves it, and does it deduct proportionally when it doesn't."

### Results

| # | Note | Score |
|---|---|--:|
| 1 | Wiki-Link Resolution Patterns | 98 |
| 2 | Frontmatter Field Checklist | 100 |
| 3 | Bridge Protocol Worked Example | 98 |
| 4 | Tag Taxonomy Conventions | 98 |
| 5 | Dedup Search Strategies | 100 |
| 6 | Hub Note Selection Criteria | 100 |
| 7 | Oversight Report Template | 100 |
| 8 | Note Enrichment Techniques | 100 |
| 9 | Relation Verb Selection | 100 |
| 10 | Archive Without Orphaning | 100 |
| **Mean** |  | **99.2** |

### Deduction pattern (the interesting part)

Three of the ten dropped to 98 (−2 each). All three deductions fell on hub-structure or classification accuracy:

- **T1 (−2 Gate 4):** proposed hub section "Instructions / Protocols" — the `Index – Agent KB` hub has no such section. Curator placed link under the existing "Core docs" section and deducted proportionally.
- **T3 (−2 Gate 6):** proposed hub section "Patterns" — the `Index – Reference Patterns` hub has no literal section by that name. Curator created a new "Curation & Meta Patterns" section.
- **T4 (−2 Gate 1):** `type: reference` submitted into `instructions/` directory. Defensible as a consumer-facing procedural reference, but not strictly aligned.

After T3, the submitter changed strategy: `read_note` on the hub first to get actual section names, then propose accordingly. T5–T10 all scored 100. **The same agent, applying lessons from each deduction, earned strictly higher scores over time.** The rubric discriminates, the feedback is actionable, and the system rewards learning.

### What the curator did well
- **Re-ran dedup independently.** Never trusted the submitter's evidence; ran its own 4 searches on every submission. No false-negative dedup hits got through.
- **Caught hub-section fabrications.** The three deductions were all for naming sections that did not exist. The submitter's other errors would have been caught too, had there been any.
- **Verified wiki-link resolution.** Every relation target (`Implements [[X]]`, etc.) was independently resolved in the graph; missing targets would have been flagged but none were.
- **Proportional deductions.** −2 for a minor hub-structure miss, not a rejection. The rubric's minimum passing threshold (91) accommodates small imperfections; rejections are reserved for structural failures.
- **Explicit advisories.** After T1, the curator's response included an actionable advisory for future submissions: "verify proposed hub section names against the actual hub structure before submission." The submitter applied the advisory and improved scores.

### Cleanup verification
All 10 accepted notes were deleted by the curator after the test. Hub links were swept from `indices/index-agent-kb` (9 entries) and `reference/patterns/index-reference-patterns` (1 entry + the new section header the curator had created). Final verification query: `search_notes(query="KB Test —", project="agent-kb")` returned 0 exact matches. The archive-without-orphaning procedure worked on its first real application.

## Non-obvious lessons

### 1. Tool access is the only reliable enforcement layer
Prompt-level instructions ("do not write to the KB without approval") are ignored under pressure. Tool allowlists ("you literally cannot call this tool") are not. Every structural constraint in a multi-agent system should be enforceable at the tool boundary if the cost of violation is high.

### 2. Independent re-verification is the point
A curator that trusts the submitter's dedup evidence provides no additional signal over the submitter's self-assessment. The value of the curator is that it re-runs the checks independently — the architecture only works if this is non-negotiable.

### 3. Proportional deductions beat binary gates
A minor Gate 4 or Gate 6 error costs 2 points, not a rejection. This keeps well-intentioned submissions flowing while still rewarding precision. The minimum passing threshold (91) is high enough to require real quality, low enough to tolerate small imperfections. A binary pass/fail on every gate would create frustrating bounce-back loops without meaningfully improving outcomes.

### 4. Hub-structure verification is the most fragile step
Three of three non-perfect scores came from proposing hub sections that did not exist. The submitter fix was simple (read the hub first) but not obvious from the protocol alone. This is now explicitly captured in `KB Test — Hub Note Selection Criteria` (before that note was deleted as part of the test — the insight survived in this retrospective and should be folded into the Quality Protocol.)

### 5. Agent registries don't hot-reload
Creating the curator agent file did NOT make it immediately available in the same session. The agent registry loads at session start. This creates a chicken-and-egg problem for first-time deployment: the curator doesn't exist yet, but only the curator can write to the KB. The bootstrap exception resolves this, but it's a one-shot — every subsequent deployment can and should route through the curator.

### 6. The retrospective itself should be a KB note
If this retrospective lives only in `/tmp` or in chat history, its lessons die with the session. Submitting it to the KB (via the curator, naturally) is the only way to make it durable. This note is that submission.

## Open gaps

- **Curator has no own KB note.** The `curator` agent file exists on disk but no corresponding KB note describes it from a retrieval perspective. The `[[curator]]` wiki-link in the Two-Fork Skill Reference note remains unresolved. Recommended next action: create an agent-reference KB note for the curator.
- **Hub-section verification should be codified.** Current guidance lives across three notes; consolidate into the Quality Protocol or create a dedicated pre-submission checklist note.
- **No rejection test run.** All 10 tests were intentionally well-composed. A future validation should include deliberately flawed submissions to verify the curator issues proper rejection reports and the submitter's resubmit loop works end-to-end.
- **Bootstrap exception is undocumented in protocols.** It happened once, successfully, but if a future session redeploys the curator the same exception will be needed. Consider documenting the bootstrap procedure explicitly.

## Relations
- Implements [[KB Submission Quality Protocol]]
- Related to [[Two-Fork Documentation — Skill Reference]]
- Derived from [[Curator Handover Knowledge]]
- Extends [[Document Curation Metaprompt]]