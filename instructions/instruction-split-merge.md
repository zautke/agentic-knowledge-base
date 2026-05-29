---
title: Instruction – Split or Merge
created: 2026-01-01
modified: 2026-01-01
type: instruction
entity_type: instruction
status: evergreen
tags:
- project/agent-kb
- op/split-merge
- status/evergreen
- source/largo
- machine/largo
permalink: agent-kb/instructions/instruction-split-merge
---

# Instruction – Split or Merge

## Goal
Control note granularity: split bloated notes, merge duplicates, canonize.

## Inputs
- Candidate note(s)
- Reason: bloat | duplicates | unclear ownership

## Steps (Split)
1. Identify separable subtopics.
2. Create new notes for each subtopic.
3. Move content into new notes.
4. Add links back to original as a hub or summary.
5. Update indices to point to new canonical notes.

## Steps (Merge)
1. Pick canonical note.
2. Copy unique content from duplicates into canonical.
3. Mark duplicates deprecated:
   - status: deprecated
   - link to canonical replacement

## Output
A cleaner, more composable knowledge base.

## Failure modes
- Too-fragmented notes: re-merge into a hub
- Broken navigation: ensure indices updated

## Next actions
Refile/Rename → Hygiene/Compression

<!-- generated_at: 2026-01-26T22:44:19Z -->