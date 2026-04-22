---
title: Curator Submission Telemetry
type: reference
permalink: agent-kb/reference/curator-submission-telemetry
created: 2026-04-19
modified: 2026-04-19
entity_type: reference
status: evergreen
tags:
- project/agent-kb
- domain/curation
- domain/observability
- pattern/append-only-log
---

# Curator Submission Telemetry

## Purpose

Append-only log of every submission the Curator has scored. One row per scoring event. It exists so that cumulative patterns — repeat deduction causes, systemic hub-structure drift, rubric calibration drift — become visible over time. A single submission's rejection report cannot surface those patterns on its own; only the aggregate can. This log is strictly append-only: never edit prior rows, never retroactively adjust scores, never collapse history. Resubmissions always earn a new row.

**Note on deleted submissions:** The ten `KB Test —` notes were deleted after the curator acceptance test per the test protocol. Their telemetry rows remain as historical scoring records even though the notes themselves no longer exist in the KB.

## How to read this log

- One row per scoring event — accepted and rejected alike.
- Resubmissions get their own row; never overwrite prior attempts.
- Score columns are raw gate points: G0 (0–10), G1 (0–10), G2 (0–10), G3 (0–20), G4 (0–25), G5 (0–10), G6 (0–15), total (0–100). Passing floor is 91.
- Deduction cause and advisory are one-liners; full analysis lives in the rejection report, not here.

## Submissions

| Date | Submission | Dir | Score | G0 | G1 | G2 | G3 | G4 | G5 | G6 | Outcome | Deduction cause | Advisory |
|------|-----------|-----|------:|---:|---:|---:|---:|---:|---:|---:|---------|-----------------|----------|
| 2026-04-19 | KB Test — Wiki-Link Resolution Patterns | instructions/ | 98 | 10 | 10 | 10 | 20 | 23 | 10 | 15 | accepted | Proposed non-existent hub section "Instructions / Protocols" | Verify hub section names against actual hub structure before submission |
| 2026-04-19 | KB Test — Frontmatter Field Checklist | instructions/ | 100 | 10 | 10 | 10 | 20 | 25 | 10 | 15 | accepted | none | none |
| 2026-04-19 | KB Test — Bridge Protocol Worked Example | reference/patterns/ | 98 | 10 | 10 | 10 | 20 | 25 | 10 | 13 | accepted | Proposed hub section "Patterns" not present verbatim in Reference Patterns index | Read the hub note before naming a section |
| 2026-04-19 | KB Test — Tag Taxonomy Conventions | instructions/ | 98 | 10 | 8 | 10 | 20 | 25 | 10 | 15 | accepted | type: reference placed in instructions/ directory | Align type with directory convention or vice versa |
| 2026-04-19 | KB Test — Dedup Search Strategies | instructions/ | 100 | 10 | 10 | 10 | 20 | 25 | 10 | 15 | accepted | none | none |
| 2026-04-19 | KB Test — Hub Note Selection Criteria | instructions/ | 100 | 10 | 10 | 10 | 20 | 25 | 10 | 15 | accepted | none | none |
| 2026-04-19 | KB Test — Oversight Report Template | instructions/ | 100 | 10 | 10 | 10 | 20 | 25 | 10 | 15 | accepted | none | none |
| 2026-04-19 | KB Test — Note Enrichment Techniques | instructions/ | 100 | 10 | 10 | 10 | 20 | 25 | 10 | 15 | accepted | none | none |
| 2026-04-19 | KB Test — Relation Verb Selection | instructions/ | 100 | 10 | 10 | 10 | 20 | 25 | 10 | 15 | accepted | none | none |
| 2026-04-19 | KB Test — Archive Without Orphaning | instructions/ | 100 | 10 | 10 | 10 | 20 | 25 | 10 | 15 | accepted | none | none |
| 2026-04-19 | Curator Agent Deployment Retrospective 2026-04-19 | reference/ | 99 | 10 | 9 | 10 | 20 | 25 | 10 | 15 | accepted | reference/ directory could be sharper (reference/retrospectives/) | Consider retrospectives subfolder for future migration pass |

## Relations

- governed_by [[KB Submission Quality Protocol]]
- observes [[Curator Handover Knowledge]]
- complements [[Curator Agent Deployment Retrospective — 2026-04-19]]