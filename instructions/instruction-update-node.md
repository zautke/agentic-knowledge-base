---
title: Instruction – Update Node
created: 2026-01-01
modified: 2026-01-01
type: instruction
entity_type: instruction
status: evergreen
tags:
- project/agent-kb
- op/update-node
- status/evergreen
permalink: agent-kb/instructions/instruction-update-node
---

# Instruction – Update Node

## Goal
Modify an existing note without breaking its identity, structure, or links.

## Inputs
- Target note identifier (title/permalink/path)
- Edit intent: append | prepend | replace section | find/replace

## Steps
1. Read the note fully before editing.
2. Choose the smallest safe edit operation:
   - Append for additive knowledge
   - Replace Section for structured updates
   - Find/Replace for repeated normalization
3. Preserve frontmatter schema:
   - do not delete , , 
4. Keep edits atomic:
   - one coherent change per update pass
5. Add links when introducing new entities.

## Output
A refined note with better accuracy and connectivity.

## Failure modes
- Accidental drift in meaning: revert/repair with a follow-up edit
- Growing bloat: use Split/Merge
- Contradictions: add "Contradicts" section + link competing notes

## Next actions
Relate Nodes → Hygiene/Compression

<!-- generated_at: 2026-01-26T22:44:19Z -->