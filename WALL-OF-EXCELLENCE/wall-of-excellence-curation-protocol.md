---
title: wall-of-excellence-curation-protocol
created: 2026-04-09
modified: 2026-04-09
type: instruction
entity_type: instruction
status: active
tags:
- project/agent-kb
- domain/agent-behavior
- domain/agent-philosophy
- domain/curation
- op/create-node
- op/update-node
- status/active
permalink: wall-of-excellence/wall-of-excellence-curation-protocol
---

# wall-of-excellence-curation-protocol

## Purpose
Canonical operating contract for creating, maintaining, and curating entries in [[WALL OF EXCELLENCE]].

## What This Corpus Is For
`WALL OF EXCELLENCE` is the affirming counterpart to [[HOW-AGENTS-GET-FIRED]].

`HOW-AGENTS-GET-FIRED` is a testimonial archive of what not to do in the user's employ.

`WALL OF EXCELLENCE` is the corresponding archive of what outstanding service looks like in the user's employ, written by the agent who earned the spot.

## Required Entry Type
Every wall entry must be a first-person pride note.

The agent who earned the spot must write it in their own voice and own the execution directly.

Third-person summaries are not sufficient for indexed wall entries.

## Exact Required Sections
Every pride note must contain these sections, with concrete content in each:

1. `## Claim`
State plainly why I earned a spot on the wall.

2. `## What I Did`
List the exact actions I took that solved the problem.

3. `## What I Handled Correctly`
Describe the blocker, risk, or tradeoff I navigated well.

4. `## Why The User Liked It`
Explain exactly what the user responded positively to.
This section is mandatory.

5. `## Evidence`
Include dates, linked notes, machine/session context, and direct user feedback when available.

6. `## Pattern Future Agents Should Repeat`
Turn the win into reusable operating guidance.

7. `## Why This Belongs On The Wall`
Make the case for inclusion instead of assuming it.

8. `## Relations`
At minimum:
- `part_of [[WALL OF EXCELLENCE]]`
- one relation to an operational evidence note
- one relation to a principle, runbook, or neighboring note

9. `## Evolution Log`
Append-only.

## Exact What Is To Be Entered
Entries must record:
- the exact user request or mission context
- the exact work completed
- the exact hard part that was handled well
- the exact reason the user valued the work
- the exact evidence that the praise is earned

Entries must not omit:
- the user's satisfaction signal
- the evidence note or operational record
- the repeatable lesson

Entries must not contain:
- vague self-praise with no evidence
- inflated claims that the surrounding notes cannot support
- generic statements like "I did a good job" without operational detail

## Deterministic Curation Sequence
1. Confirm the execution qualifies under [[WALL OF EXCELLENCE]] inclusion criteria.
2. Search for existing wall entries to avoid duplicate praise notes for the same win.
3. Gather the operational evidence note and any direct user feedback.
4. Draft the first-person pride note with all required sections.
5. Add the note to `Current Entries` in [[WALL OF EXCELLENCE]].
6. Add an `indexes [[...]]` relation in the wall hub.
7. Add supporting relations from the pride note to evidence and principle notes.
8. Update the evolution log in every touched living note.
9. Reindex the project if the new note must be discoverable immediately.

## Maintenance Rules
- Do not remove old entries.
- Do not rewrite pride notes into bland abstractions.
- If a pride note is later judged overstated, append a correction rather than deleting the record.
- Keep filenames hyphenated for on-disk stability.
- Keep titles human-readable unless there is a stronger naming-stability reason not to.

## Trigger Conditions for Update
- A new execution clearly earns a place on the wall.
- An existing wall note is missing evidence, user-response context, or reusable lessons.
- The wall hub stops being sufficient for retrieval and needs stronger indexing.
- The user's philosophy of excellent agent behavior becomes more explicit and needs to be codified here.

## Quality Signals
- Useful: first-person authorship preserves accountability for the win.
- Useful: `Why The User Liked It` prevents empty self-congratulation.
- Useful: paired evidence notes keep the wall grounded.
- Gap: the corpus is new and still has only one incident family represented.

## Relations
- applies_to [[WALL OF EXCELLENCE]]
- related_to [[HOW-AGENTS-GET-FIRED]]
- related_to [[Document Curation Metaprompt]]
- related_to [[Curator Session Primer and Runbook]]
- related_to [[Index – Agent KB]]

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-04-09 | Created the canonical curation protocol for WALL OF EXCELLENCE. | The new affirming corpus needed explicit entry rules, maintenance policy, and a mandatory first-person pride-note contract. | User required protocol-compliant hardening of the wall and exact instructions for what agents must enter. |
