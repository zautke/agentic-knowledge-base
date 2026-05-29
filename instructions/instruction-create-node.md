---
title: Instruction – Create Node
created: 2026-01-01
modified: 2026-01-01
type: instruction
entity_type: instruction
status: evergreen
tags:
- project/agent-kb
- op/create-node
- status/evergreen
- source/largo
- machine/largo
permalink: agent-kb/instructions/instruction-create-node
---

# Instruction – Create Node

## Goal
Create a new durable memory node (note) with correct type, metadata, and placement.

## Inputs
- Intended concept (what is the note *about*)
- Expected lifecycle (, , )
- Folder (taxonomy fit)
- Minimal tags

## Steps
1. Choose **entity_type**: fundamental | reference | instruction | pattern | example | source | index | scratch
2. Choose filename + title:
   - Use stable, descriptive naming
   - Prefer verb-noun for instructions ("Instruction – ...")
3. Write frontmatter:
   - include , , , , 
4. Write body:
   - put the *purpose* in first 2–3 lines
   - include explicit links to known hubs
5. If it is a new core concept:
   - link it into a relevant Index or Fundamental hub

## Output
A searchable, typed, linked note.

## Failure modes
- Wrong folder: fix with Refile/Rename
- Missing tags: add  + domain/op tags
- Duplicates: handle with Split/Merge

## Next actions
Relate Nodes → Create Index/Hubs

<!-- generated_at: 2026-01-26T22:44:19Z -->