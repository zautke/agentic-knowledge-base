---
title: Instruction – Deprecate or Archive
created: 2026-01-01
modified: 2026-01-01
type: instruction
entity_type: instruction
status: evergreen
tags:
- project/agent-kb
- op/deprecate-archive
- status/evergreen
permalink: agent-kb/instructions/instruction-deprecate-archive
---

# Instruction – Deprecate or Archive

## Goal
Retire incorrect or obsolete notes safely without losing history.

## Inputs
- Target note
- Replacement canonical note (if any)
- Reason for deprecation

## Steps
1. Change metadata:
   - 
2. Add a visible banner at top:
   - "Deprecated: replaced by [[X]]"
3. Keep original content (do not delete).
4. Update indices to point to replacement.

## Output
A stable knowledge base with clear truth routing.

## Failure modes
- No replacement exists: create a new canonical reference first

## Next actions
Hygiene/Compression

<!-- generated_at: 2026-01-26T22:44:19Z -->