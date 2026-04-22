---
title: Instruction – Relate Nodes
created: 2026-01-01
modified: 2026-01-01
type: instruction
entity_type: instruction
status: evergreen
tags:
- project/agent-kb
- op/relate-nodes
- status/evergreen
permalink: agent-kb/instructions/instruction-relate-nodes
---

# Instruction – Relate Nodes

## Goal
Create meaningful edges between notes to improve navigation + retrieval.

## Inputs
- Source note
- Target note(s)
- Relation type: explains | depends_on | implements | used_by | derived_from | related

## Steps
1. Add a **direct wiki link** from source → target:
   - 
2. Prefer an explicit relation section:
   - "Explains", "Depends on", etc.
3. Ensure edges are non-redundant:
   - avoid linking everything to everything
4. If a note becomes a hub:
   - consider creating an Index note for it

## Output
A graph edge that improves multi-hop retrieval.

## Failure modes
- Too many weak links: prune
- Missing hub: create Index/Hubs

## Next actions
Create Index/Hubs → Hygiene/Compression

<!-- generated_at: 2026-01-26T22:44:19Z -->