---
title: Two-Fork Documentation — Skill Reference
type: reference
permalink: agent-kb/instructions/two-fork-documentation-skill-reference
created: 2026-04-19
modified: 2026-04-19
entity_type: reference
status: evergreen
tags:
- project/agent-kb
- domain/curation
- domain/documentation
- op/protocol
- source/largo
- machine/largo
---

# Two-Fork Documentation — Skill Reference

A Claude Code skill that produces two synchronized artifacts for every documentation request: a filesystem file at the requested path, and a curated KB note submitted through the `curator` agent.

## Purpose

Ensures that every documentation artifact written to the filesystem is simultaneously registered in the knowledge base as a well-curated, retrievable, graph-linked note. The filesystem and KB are kept in sync by design — one triggers the other. Neither fork is optional.

## What the Skill Does

When invoked for any documentation task (README, runbook, API reference, ADR, protocol, tutorial, changelog), the skill:

1. Writes the filesystem artifact as requested (no change to normal behavior)
2. Prepares a KB submission package: full note content + hub target + 4-step dedup evidence + Gate 6 oversight report
3. Spawns the `curator` agent to independently score and write the KB note
4. Reports the curator's decision — approval (with permalink + hub update) or rejection (with gate-level feedback requiring resubmission)

**The skill does not write to the KB directly.** `write_note` and `edit_note` are not in its `allowed-tools`. The `curator` agent holds exclusive write authority.

## File Location

`~/.claude/skills/two-fork-documentation/SKILL.md`

## Tool Access (Read-Only)

```
allowed-tools: mcp__kb__search_notes, mcp__kb__read_note,
               mcp__kb__build_context, mcp__kb__list_directory
```

No `write_note` or `edit_note`. Enforced at the tool level.

## Submission Workflow

```
Pre-check scaffold → 4-step dedup → Choose type/directory →
Compose note → Identify hub target → Compile submission package →
Spawn curator → Handle approval or rejection
```

Task is not complete until the curator issues an approval. Rejections (score ≤ 90/100) require fixing the specific gate issues and resubmitting.

## Quality Bar

Every KB submission is scored on the 7-gate rubric ([[KB Submission Quality Protocol]]). Minimum: **91/100**. Gates 4 (Linking, 25 pts) and 6 (Oversight Report, 15 pts) together = 40% of the score. An undocumented orphan note cannot pass.

## Document Type → KB Note Type Mapping

| Filesystem Doc | KB `type` | KB Directory |
|---|---|---|
| README / overview | `reference` | `projects/<slug>/` |
| Runbook | `reference` | `projects/<slug>/operations/runbooks/` |
| API reference | `reference` | `projects/<slug>/knowledge/` |
| Architecture doc | `reference` | `projects/<slug>/architecture/` |
| ADR | `reference` | `projects/<slug>/architecture/decisions/` |
| Protocol | `instruction` | `projects/<slug>/protocols/` |
| Tutorial | `example` | `projects/<slug>/knowledge/use-cases/` |
| Global pattern | `pattern` | `reference/patterns/` |

## Relations

- implements [[KB Submission Quality Protocol]]
- related_to [[Codebase KB Ingestion Protocol]]
- related_to [[Curator Handover Knowledge]]
- related_to [[Document Curation Metaprompt]]