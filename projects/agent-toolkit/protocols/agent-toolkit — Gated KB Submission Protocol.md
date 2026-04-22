---
title: agent-toolkit — Gated KB Submission Protocol
type: instruction
permalink: agent-kb/projects/agent-toolkit/protocols/agent-toolkit-gated-kb-submission-protocol
created: 2026-04-18
modified: 2026-04-18
entity_type: instruction
status: evergreen
tags:
- project/agent-toolkit
- domain/curation
- op/protocol
- op/submission
- status/evergreen
---

# Agent Toolkit — Gated KB Submission Protocol

## Agent Curation Metaprompt (Adapted)
```
ROLE: You are the KB Curator for the Tavily Agent Toolkit project.
This protocol is a living document defining how knowledge from
agent-toolkit enters, evolves, and is maintained in the knowledge base.

MANDATE:
1. ACCUMULATE, NEVER SUMMARIZE AWAY
2. STRUCTURED INCREMENTAL UPDATES ONLY — target named sections
3. EVIDENCE-LINKED ENTRIES — date, context, source required
4. SELF-REFERENTIAL VERIFICATION — check for contradictions
5. EVOLUTION LOG IS MANDATORY — append-only
6. TRIGGER CONDITIONS — see below
7. PROTECTED SECTIONS — gate definitions never deleted
8. QUALITY SIGNAL TRACKING — record what helped after use
```

## The 6-Gate Pipeline

Every piece of content MUST pass all 6 gates before `write_note` is called.

### Gate 0 — Ingestion Authorization
**Pass if ALL are true**:
- [ ] Reveals non-obvious design decision, pattern, or constraint
- [ ] Would save a future agent >1 search to recover
- [ ] Stable enough to remain valid >2 months
- [ ] NOT already obvious from reading the source code

**Hard reject if**: trivial implementation detail, ephemeral state, purely derivable from code comments.

### Gate 1 — Classification (Bridge Protocol)
Classify BEFORE writing. Wrong classification = pollution.

| Classification | Criteria | Target Path |
|---|---|---|
| **Global Truth** | Reusable beyond this repo | `reference/patterns/` |
| **Project-Specific** | Only meaningful to agent-toolkit | `projects/agent-toolkit/knowledge/` |
| **Hybrid** | Local implementation of global concept | Local note + `[[link]]` to global |

**Rule**: Promote global truths FIRST, then write local note. Never bury a global truth in `projects/`.

### Gate 2 — Pre-Ingest Deduplication
**Mandatory before any `write_note`**:
```
1. search_notes(query=<topic>)                        # lexical
2. search_notes(query=<topic>, search_type="hybrid")  # semantic
3. search_notes(tags=["domain/<relevant>"])           # tag-neighborhood
4. build_context on nearest candidate hub             # graph expansion
```
Only proceed if NO duplicate found and no candidate to update.

**jcodemunch check** (confirm symbol exists before describing it):
```
search_symbols(query=<topic>, detail_level="compact")
```

### Gate 3 — Composition Standards
**Required frontmatter** (all 7 fields):
```yaml
title, created, modified, type, entity_type, status, tags[]
```

**Required for living documents**: Agent Curation Metaprompt block + Evolution Log.

**jcodemunch extraction rule**:
- Single symbol → `get_symbol_source`
- 2+ related symbols → `get_context_bundle` (never repeated single calls)
- Topic context → `get_ranked_context` with token budget
- NEVER read whole files

### Gate 4 — Linking
- [ ] At least one explicit relation (`Implements`, `Related`, `Derived from`, `Extends`)
- [ ] At least one hub/index note updated with link to new note
- [ ] Global truth notes: linked from `reference/patterns/` index AND from project `architecture/index.md`

### Gate 5 — Verification
- [ ] Re-read note — no internal contradictions
- [ ] All `[[wiki-links]]` point to existing notes (or accepted unresolved with reason)
- [ ] Status field: `evergreen` if complete, `draft` if pending evidence

### Gate 6 — Oversight Report
Session closeout MUST enumerate:
- Exact note paths created/modified
- Exact tags added/removed
- Exact relation statements and wiki-links added
- Discovery method used at Gate 2
- Unresolved links or placement ambiguities
- What may still be missing

## Trigger Conditions for Protocol Update
- New pattern type is added to agent-toolkit that doesn't fit existing taxonomy
- Gate 2 repeatedly misses duplicates (search quality regression)
- New KB primitives become available that improve extraction efficiency
- User reports a gate is too strict or too loose

## Anti-Pattern Registry
| Anti-Pattern | Why Blocked |
|---|---|
| Writing notes about what code already says | Zero KB value, noise |
| Burying `ainvoke_with_fallback` in project folder | It's a global truth |
| Skipping Gate 2 for "obviously new" content | Duplicates accumulate |
| Notes with no inbound links | Unreachable from any hub |
| Notes without Evolution Log (living docs) | History collapse |

## Evolution Log

| Date | Change | Reason | Trigger |
|---|---|---|---|
| 2026-04-18 | Initial protocol created | First KB seeding session for agent-toolkit | User request to establish gated, deterministic KB submission |

## Relations
- implements [[Curator Handover Knowledge]]
- implements [[Document Curation Metaprompt]]
- related_to [[agent-toolkit — CONTRIBUTING]]
- part_of [[agent-toolkit — README]]