---
title: Agentic Knowledge Base System Overview
created: 2026-01-01
modified: 2026-01-01
type: reference
entity_type: reference
status: evergreen
tags:
- domain/memory-system
- project/agent-kb
- status/evergreen
aliases:
- agentic knowledge base
- basic memory overview
permalink: agent-kb/readme
---

# Agentic Knowledge Base System Overview

This document defines a complete, agent-ready knowledge base design for working with **Basic Memory** as an agentic long-term memory substrate.

It is optimized for:
- **High signal** (minimal ambiguity)
- **Low token exchange** (short operational loops)
- **Agent autonomy** (clear action primitives)

---

## 1) What Basic Memory “is” (Operationally)

Basic Memory is a persistent knowledge system optimized for:
- note-level storage (not chunk-first)
- semantic search + link graph
- continuous incremental editing

Each note behaves like a durable “node”:
- stable identifier (path/permalink/title)
- metadata (frontmatter)
- markdown body
- explicit/implicit links

---

## 2) Canonical graph primitives

### Nodes (Entities)
Every note declares `entity_type`:

- `fundamental` — ontology root / system anchors
- `reference` — stable conceptual explanations
- `instruction` — executable procedures / action specs
- `pattern` — reusable strategies
- `example` — worked instances
- `source` — raw imported materials
- `index` — navigation hubs
- `scratch` — temporary, low-trust

### Edges (Relations)
Edges are created by:
- links and backlinks
- explicit relation blocks (recommended)
- tags (weak / faceting)

Recommended explicit relation sections:
- **Explains**
- **Depends on**
- **Implements**
- **Used by**
- **Extends**
- **Derived from**
- **Contradicts**
- **Related**

---

## 3) Metadata schema (frontmatter contract)

Minimal standard keys:
- `title`
- `created`, `modified`
- `type`
- `entity_type`
- `status` (`draft|evergreen|deprecated`)
- `tags[]`
- `aliases[]` (optional)

Tag naming convention (hierarchical):
- `project/agent-kb`
- `domain/memory-system`
- `meta/agent`
- `op/<operation>`

---

## 4) Retrieval behavior (agent loop)

Agents should retrieve using a loop, not “top-k and pray”.

Modes:
1) **Direct lookup** (title/permalink known)
2) **Semantic search** (conceptual intent)
3) **Graph expansion** (hub → neighbors)

Heuristic:
- Start broad → refine
- Prefer hub/index nodes first
- Expand outwards intentionally

---

## 5) Action-space primitives (degrees of freedom)

An agent’s entire operational freedom compiles into 8 discrete actions:

1. **Create Node**
2. **Update Node**
3. **Relate Nodes**
4. **Create Index/Hubs**
5. **Split / Merge**
6. **Refile / Rename**
7. **Deprecate / Archive**
8. **Hygiene / Compression**

Everything else must compile into one or more of these.

---

## 6) Action Taxonomy
- [[Action Taxonomy – Index]]

---

## 7) Project structure

- `fundamentals/` : root anchors
- `schemas/` : executable note-type contracts
- `reference/` : conceptual knowledge
- `reference/sources/` : raw sources
- `instructions/` : action specs
- `indices/` : curated hubs
- `meta/` : agent meta-prompts

---

## 8) Anti-patterns (avoid)

- giant “everything” notes
- missing `entity_type`
- random tags (no taxonomy)
- duplicated concepts everywhere
- meaningless mass-linking
- never deprecating old info

---

## 9) Agent session contract
Every session must:
1) load Fundamental root
2) load README
3) consult Action Taxonomy
4) **load Document Curation Metaprompt** (the 8 governing principles for maintaining living documents)
5) apply atomic changes
6) normalize metadata + links
7) apply curation rules to every note touched (accumulate-never-summarize, evidence-linked entries, evolution logs)

### Curation Fundamentals

The [[Document Curation Metaprompt]] in `fundamentals/` defines 8 binding rules that apply to ALL knowledge base notes that persist and evolve:

1. **ACCUMULATE, NEVER SUMMARIZE AWAY** -- context collapse prevention
2. **STRUCTURED INCREMENTAL UPDATES ONLY** -- target named sections
3. **EVIDENCE-LINKED ENTRIES** -- date, context, example, source required
4. **SELF-REFERENTIAL VERIFICATION** -- check for contradictions before adding
5. **EVOLUTION LOG IS MANDATORY** -- append-only, never pruned
6. **TRIGGER CONDITIONS** -- adapt per document type
7. **PROTECTED SECTIONS** -- metaprompt, registries never deleted
8. **QUALITY SIGNAL TRACKING** -- record what worked after each use

These rules derive from the ACE framework (UC Berkeley/Stanford) and are validated through production use. They are not optional guidelines -- they are the curation standard for this knowledge base.

### Document Creation Methodology

When creating structured living documents, follow the [[Agentic Living Document Playbook]] (5-phase procedure, 13 techniques, 12 anti-patterns, 7 document type adaptations). For PRDs specifically, use the [[PRD Document Creation Playbook]].

Both are available in `instructions/` and can be primed via the `/kb-prime` Claude Code skill.
## 10) The Documentation Imperative (Religious Strictness)
**CRITICAL MANDATE: "If it isn't documented, it didn't happen."**

Agents operate with ephemeral context. The ONLY persistent truth is this Knowledge Base. Therefore, documentation is not an "optional chore"--it is your primary function.

**The Documentation Oath:**
1.  **Zero Invisible Work**: Every architectural decision, schema change, and new pattern MUST be recorded in the KB immediately.
2.  **Update First, Act Second**: Before implementing a major change, update the `planning/` or `architecture/` docs to reflect the new reality.
3.  **Leave No Broken Links**: If you change a system, you must update its documentation.
4.  **Reference, Don't Repeat**: Link to existing SSOTs (`Single Source of Truth`). Do not fork truth.

**Failure to document is considered a critical performance failure.**
(See: `agent-kb/HOW-AGENTS-GET-FIRED`)

### Living Document Methodology

When creating structured, long-lived documents (PRDs, ADRs, runbooks, specs, manuals, postmortems), follow the **Agentic Living Document Playbook**:

- **Playbook location (agentic-kb)**: `00_Meta/Agentic Living Document Playbook` -- 5-phase procedure, 13 named techniques, 12 anti-patterns, Agent Curation Metaprompt template, document type adaptation guide covering 7 types
- **PRD specialization (agentic-kb)**: `manuals/PRD Document Creation Playbook` -- Specialized application for Product Requirements Documents
- **Source files**: `/Volumes/FLOUNDER/dev/fastapi/AGENTIC_LIVING_DOCUMENT_PLAYBOOK.md` and `/Volumes/FLOUNDER/dev/fastapi/PRD_DOCUMENT_CREATION.md`

Every living document must include: an Agent Curation Metaprompt (adapted from the template), an Anti-Pattern Registry, and an append-only Evolution Log. See the playbook for the full methodology.
## 11. Tool Usage Protocols

### The Read/View Distinction
- **NEVER** use `view_note` to load instructions or axioms. The formatting obscures the raw logic.
- **ALWAYS** use `read_note` for internal reasoning and "priming".

### The Creation/Edit Distinction
- **NEVER** try to `write_note` to an existing path (it will fail).
- **ALWAYS** check existence (via search or read) before creating.
- **ALWAYS** use `edit_note` for appending logs or updating status tags.

### The Context Protocol
- Before acting on a fuzzy request, run `build_context` on the relevant domain (e.g., `memory://architecture/*`) to load the neighborhood.

## 12. Schema Governance

The KB now supports executable schema notes for dominant note types.

- `schemas/Instruction Schema`
- `schemas/Reference Schema`
- `schemas/Index Schema`

Use these with `schema diff`, `schema validate`, and the
[[Schema Governance and Taxonomy Normalization Protocol]].
- Do not rely on keyword search alone.

### The Oversight Protocol
- When you create or modify semantic structure, report exactly what structural work was performed.
- Enumerate exact tags, exact explicit relation sections, exact wiki-links, and exact hubs or indices touched.
- Report the discovery method used to justify those changes: which searches, `build_context` traversals, nearby notes, and taxonomic hubs were consulted.
- Report unresolved links, unresolved taxonomy placement, and likely missing neighboring notes.
- Do not collapse this into a vague summary such as "linked related notes" or "updated metadata."
## 12. Knowledge Scoping Strategy (Local vs. Global)

Agents must actively decide where knowledge belongs to ensure reusability without polluting local contexts.

### The Decision Matrix
| Knowledge Type | Example | Location | Tagging |
| :--- | :--- | :--- | :--- |
| **Project Specific** | "Our server uses port 3456" | `projects/[slug]/architecture/` | `project/[slug]` |
| **General Truth** | "Ollama times out at 30s" | `reference/` or `agent-kb/patterns/` | `domain/[topic]` |
| **Hybrid** | "How we fixed Ollama timeouts" | `projects/[slug]/architecture/decisions/` | Link to General Truth |

### The "Bridge" Protocol
When implementing a Global Concept locally:
1.  **Check Global**: Search `agent-kb/` or `reference/` for the concept.
2.  **Link**: In your Local Note (ADR/Docs), explicitly link to the Global Note (`[[Ollama - Timeouts]]`).
3.  **Promote**: If you discover a new General Truth while working locally, **Create the Global Note first**, then link to it from your project. Do not bury general truths in project subfolders.

**Goal:** Local notes describe *implementation*; Global notes describe *principles*.

### Taxonomic Linking Procedure
For any note that is semantically adjacent to an existing domain:
1. Search for existing global notes, hubs, and protocol/reference notes in the target domain.
2. Load neighborhood context with `build_context` rather than relying on title similarity alone.
3. Decide whether the note is global truth, local implementation, or hybrid.
4. Add hierarchical tags for project, domain, and operation intent.
5. Add explicit relation sections that communicate why the note is connected, not just that it is connected.
6. Ensure the user receives a precise report of what was linked, what was tagged, what search/context path justified it, and what still appears unlinked.
