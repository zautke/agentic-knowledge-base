---
title: Instruction – Hygiene or Compression
created: 2026-01-01
modified: 2026-01-01
type: instruction
entity_type: instruction
status: evergreen
tags:
- project/agent-kb
- op/hygiene
- status/evergreen
permalink: agent-kb/instructions/instruction-hygiene-compression
---

# Instruction – Hygiene or Compression

## Goal
Keep the system coherent, compact, searchable, and agent-friendly.

## Inputs
- Set of modified notes
- Current project standards

## Steps
1. Normalize metadata:
   - ensure , ,  exist
2. Reduce redundancy:
   - remove repeated paragraphs
   - link to canonical references instead
3. Make relations explicit:
   - add "Explains/Depends/Related" sections
4. Ensure navigability:
   - at least one inbound link from an index/hub for important nodes
5. Verify idempotency:
   - changes safe to re-run without duplication

## Output
A higher-quality memory graph with strong retrieval affordances.

## Failure modes
- Overcompression (lost details): keep details in sources/examples
- Too many indices: merge or scope down

## Next actions
Create Index/Hubs → Relate Nodes

<!-- generated_at: 2026-01-26T22:44:19Z -->