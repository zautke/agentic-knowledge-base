---
title: CONTRIBUTING
type: instruction
permalink: agent-kb/projects/agent-toolkit/contributing
created: 2026-04-18
modified: 2026-04-18
entity_type: instruction
status: evergreen
tags:
- project/agent-toolkit
- domain/curation
- op/contribution
- status/evergreen
---

# Agent Toolkit KB — Contribution Rules

## The Curator's Mandate
Every agent touching this project's KB folder is operating as **Curator**. You are here to GARDEN this graph — not to dump raw content.

## The Gate Checklist (run before every `write_note`)

- [ ] **Gate 0**: Content reveals non-obvious design decision, pattern, or constraint (not just what the code already says)
- [ ] **Gate 1**: Classified as Global Truth / Project-Specific / Hybrid (see [[agent-toolkit — Gated KB Submission Protocol]])
- [ ] **Gate 2**: `search_notes` (lexical + hybrid) confirms no duplicate exists
- [ ] **Gate 3**: Note has proper frontmatter — all 7 required fields present
- [ ] **Gate 4**: At least one explicit relation written; at least one hub updated
- [ ] **Gate 5**: Self-check for contradictions + unresolved wiki-links
- [ ] **Gate 6**: Session closes with enumerated oversight report

## jcodemunch Rules
- **Always `get_file_outline` before any `get_symbol_source`**
- **Always `get_context_bundle` for 2+ related symbols** (never repeated single calls)
- **Never read whole files** — use `get_ranked_context` with a token budget
- **jcodemunch repo id**: `local/agent-toolkit-49aa8c6c`

## Anti-Patterns (hard blocks)
- Writing notes about what code already makes obvious
- Creating project-local notes for global truths (use Bridge Protocol)
- Skipping `search_notes` before `write_note`
- Notes without Evolution Log (for living documents)
- Notes with no inbound links

## Relations
- implements [[agent-toolkit — Gated KB Submission Protocol]]
- part_of [[Curator Handover Knowledge]]
- implements [[Codebase KB Ingestion Protocol]] — canonical global protocol; this CONTRIBUTING is the project-specific instantiation