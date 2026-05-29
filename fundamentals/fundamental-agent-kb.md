---
title: Fundamental – Agent KB
created: 2026-01-01
modified: 2026-05-28
type: fundamental
entity_type: fundamental
status: evergreen
tags:
- project/agent-kb
- domain/memory-system
- status/evergreen
- source/largo
- machine/largo
aliases:
- Agent KB root
permalink: agent-kb/fundamentals/fundamental-agent-kb
---

# Fundamental – Agent KB

Canonical root entity for the `agent-kb` memory project.

## Explains
- [[Agentic Knowledge Base System Overview]]

## Entry points
- [[Index – Agent KB]]
- [[Action Taxonomy – Index]]
- [[Document Curation Metaprompt]]

## Document Methodology
- [[Agentic Living Document Playbook]]
- [[PRD Document Creation Playbook]]
## Meta prompts
- [[Meta-Instruction – Add Knowledge README]]
- [[Meta-Instruction – Build Action Taxonomy]]
- [[SessionStart Primer – Agentic Knowledge Base]]

<!-- generated_at: 2026-01-26T22:44:19Z -->
## 6. Tool Capability Matrix (Operational Primitives)

Agents must select the correct tool for the intent. Do not guess.

### Reading & Context
| Intent | Tool | Note |
|--------|------|------|
| **Ingest Logic/Facts** | `kb__read_note` | Returns raw markdown. Use for reasoning. |
| **Display/Render** | `kb__view_note` | Returns formatted artifact. Use for user presentation. |
| **Load Graph Context** | `kb__build_context` | Follows links from a node to build a semantic prompt. |
| **Check History** | `kb__recent_activity` | Returns chronological changes. Use for session priming. |

### Writing & Modification
| Intent | Tool | Note |
|--------|------|------|
| **Create New** | `kb__write_note` | Fails if note exists. Requires `folder`. |
| **Update Existing** | `kb__edit_note` | Use `append` for logs, `replace` for refactors. |
| **Move/Rename** | `kb__move_note` | Updates all backlinks automatically. |
| **Delete** | `kb__delete_note` | Use sparingly. Prefer deprecation. |

### Project Management
| Intent | Tool | Note |
|--------|------|------|
| **Switch Context** | `kb__switch_project` | Sets active project for subsequent calls. |
| **Check Sync** | `kb__sync_status` | Verify indexing before searching. |

## 7. Oversight Requirement

Semantic KB work is not complete until the user has an auditable summary of the structural changes.

Required closeout contents:
- exact note paths or permalinks touched
- exact tags and relation statements changed
- exact links added or left unresolved
- exact discovery approach used to find related notes and determine taxonomy placement
- explicit statement of what may have been overlooked

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-05-28 | Added the mandatory Evolution Log and an explicit Relations block; bumped `modified` from the stale `2026-01-01` value. Section numbering left intact per Rule 7 (the orphan `## 6`/`## 7` headings are preserved to avoid breaking cross-references). | This root fundamental was a living governance doc with no append-only Evolution Log, violating Metaprompt Rule 5, and its entry-point links were maintained as ad-hoc lists rather than a typed Relations block. | Governance-doc curation sweep over `fundamentals/`. |

## Relations

- explains [[Agentic Knowledge Base System Overview]]
- has_entry_point [[Index – Agent KB]]
- has_entry_point [[Action Taxonomy – Index]]
- has_entry_point [[Document Curation Metaprompt]]
- related_to [[Curator Handover Knowledge]]
- related_to [[KB Submission Quality Protocol]]
- related_to [[Curator Session Primer and Runbook]]
