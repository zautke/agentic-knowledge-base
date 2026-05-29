---
title: Instruction – Refile or Rename
created: 2026-01-01
modified: 2026-01-01
type: instruction
entity_type: instruction
status: evergreen
tags:
- project/agent-kb
- op/refile-rename
- status/evergreen
- source/largo
- machine/largo
permalink: agent-kb/instructions/instruction-refile-rename
---

# Instruction – Refile or Rename

## Goal
Move notes into the correct taxonomy location and improve naming stability.

## Inputs
- Target note
- Destination folder
- Improved canonical title/filename

## Steps
1. Decide correct folder:
   - fundamentals/ | reference/ | instructions/ | indices/ | meta/ | sources/
2. Rename for clarity:
   - instructions use verb-object form
3. Preserve backlinks:
   - keep titles stable when possible
4. Update index references if required.

## Output
A note placed where future agents will find it quickly.

## Failure modes
- Renaming causes broken links: add aliases or update links across indices

## Next actions
Relate Nodes → Hygiene/Compression

<!-- generated_at: 2026-01-26T22:44:19Z -->