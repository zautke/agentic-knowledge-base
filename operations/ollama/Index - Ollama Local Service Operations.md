---
title: Index - Ollama Local Service Operations
type: index
permalink: operations/ollama/index-ollama-local-service-operations
created: 2026-02-18
modified: 2026-02-19
entity_type: index
status: evergreen
tags:
- project/agent-kb
- domain/ollama
- domain/operations
- domain/service-management
- index/ollama
---

# Index - Ollama Local Service Operations

## Purpose and Scope
Navigation hub for operational knowledge about local Ollama service management via `olsvc.sh`, including command surface, config drift controls, API compatibility mapping, and host audit findings.

## Canonical Notes
- [[Ollama Local Service (olsvc.sh) - Operational Session 2026-02-18]] - Session-grounded operational record for script enhancements, audit findings, and validation evidence.
- `[[Ollama Local Service (olsvc.sh) - Operational Session 2026-02-19]]` - UNRESOLVED (2026-05-28 curation check): no note with this title exists under `operations/ollama`. The 2026-02-19 Evolution Log entries below describe a planned follow-on session record that was never created. Left as a tracked gap rather than fabricated.
- `[[Ollama Local Service (olsvc.sh) - Multi-Agent Field Test Plan]]` - UNRESOLVED (2026-05-28 curation check): no note with this title exists. Planned multi-agent test-plan node was never created. Tracked gap.

## Related Infrastructure Context
- [[Machine Profile - largo]] - Host baseline where this session's operational checks were executed.
- [[Index – Agent KB]] - Top-level index entry point.

## Curation Primitive Compilation
### 2026-02-18
1. Create Node: Created this hub index note.
2. Update Node: `indices/index-agent-kb` updated to link this hub and log evolution.
3. Relate Nodes: Linked hub <-> operational record <-> machine/index anchors.
4. Create Index-Hubs: This note is the dedicated hub for Ollama service operations.
5. Split/Merge: Split session knowledge into hub + focused operational record.
6. Refile/Rename: No refile/rename required in this session.
7. Deprecate/Archive: No deprecations required in this session.
8. Hygiene/Compression: Avoided duplication by confirming no prior Ollama/olsvc notes existed before creation.

### 2026-02-19
1. Create Node: Added [[Ollama Local Service (olsvc.sh) - Operational Session 2026-02-19]] and [[Ollama Local Service (olsvc.sh) - Multi-Agent Field Test Plan]].
2. Update Node: Updated this hub and [[Index – Agent KB]] with direct discoverability links.
3. Relate Nodes: Added explicit relations from this index to both new operational nodes.
4. Create Index-Hubs: Reused this hub as canonical entrypoint rather than creating parallel hubs.
5. Split/Merge: Kept session outcomes and reusable test-plan logic in separate nodes.
6. Refile/Rename: No taxonomy moves required.
7. Deprecate/Archive: No deprecations required.
8. Hygiene/Compression: Avoided duplicating prior 2026-02-18 baseline content by linking forward.

## Trigger Conditions for Update
- New `olsvc.sh` operational session note is added under `operations/ollama`.
- Command inventory or compatibility mapping changes in canonical test-plan artifacts.
- New `/Volumes/DEV/.ollama` incident patterns require index-level rerouting to fresh diagnostics notes.

## Quality Signals
- Useful: hub + focused note split improves retrieval for future operations work.
- Useful: separate test-plan node improves reuse across recurring field-test runs.
- Gap: no pre-existing Ollama domain node in `agent-kb`; this hub now serves as initial anchor.
- Follow-up quality check: validate backlink growth from future Ollama incidents and maintenance sessions.

## Evolution Log
| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-02-18 | Created Ollama operations hub and linked canonical session note | Provide durable discoverability and non-duplicative index entry for new operational domain | Session curation for `olsvc.sh` enhancements and `.ollama` audit |
| 2026-02-19 | Added 2026-02-19 session note and dedicated multi-agent field-test plan node to canonical set | Expand operational continuity and preserve test-plan SSOT as separate reusable node | Field-test artifact ingestion and curator indexing update |
| 2026-02-19 | Expanded curation-primitive compilation and added index-level trigger conditions | Align hub note with strict curation protocol and explicit update triggers | Compliance pass for document-curation rules during same session |
| 2026-05-28 | Annotated two unresolved canonical-note links (2026-02-19 session, Multi-Agent Field Test Plan) as tracked gaps; the referenced notes were never created. Demoted matching `related_to` lines to a documented unresolved block. | Index claimed notes that do not exist on disk, producing two dangling links | KB curation hygiene sweep (operations/reference partition) |

## Relations
- related_to [[Ollama Local Service (olsvc.sh) - Operational Session 2026-02-18]]
- related_to [[Machine Profile - largo]]
- related_to [[Index – Agent KB]]

## Unresolved Relations (tracked gaps, 2026-05-28)
- related_to `[[Ollama Local Service (olsvc.sh) - Operational Session 2026-02-19]]` — note never created
- related_to `[[Ollama Local Service (olsvc.sh) - Multi-Agent Field Test Plan]]` — note never created