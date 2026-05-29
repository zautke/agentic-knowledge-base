---
title: Codebase KB Ingestion Protocol
type: instruction
permalink: agent-kb/instructions/codebase-kb-ingestion-protocol
created: 2026-04-18
modified: 2026-04-18
entity_type: instruction
status: evergreen
tags:
- project/agent-kb
- domain/curation
- op/protocol
- op/ingestion
- source/largo
- machine/largo
---

# Codebase KB Ingestion Protocol

## Agent Curation Metaprompt

```
ROLE: You are a KB Curator operating the Codebase Ingestion Protocol.
This is a living instruction document. Apply it verbatim.

MANDATE:
1. ACCUMULATE, NEVER SUMMARIZE AWAY
2. STRUCTURED INCREMENTAL UPDATES ONLY
3. EVIDENCE-LINKED ENTRIES — date, source, concrete symbol
4. SELF-REFERENTIAL VERIFICATION — check for contradictions before writing
5. EVOLUTION LOG IS MANDATORY — append-only
6. TRIGGER CONDITIONS — see below
7. PROTECTED SECTIONS — gate definitions are never deleted
8. QUALITY SIGNAL TRACKING — record what helped after each use
```

---

## Purpose and Scope

A deterministic, repository-agnostic protocol for systematically ingesting a software codebase into the `agent-kb` knowledge graph. It captures architecture, patterns, API surface, agents, use cases, operations, and integrations as reusable, linked KB notes.

**Companion skill**: `kb:codebase-ingestion` — Claude Code slash command `/kb-codebase-ingestion`

**Prerequisites**:
- `kb-prime` bootstrap sequence completed
- jcodemunch MCP available and repo indexed (`resolve_repo` → `index_folder` if needed)
- User provides: repo path, project slug (kebab-case), display name, project type

---

## The 5-Phase Process

### Phase 0: Repository Orientation

Run all jcodemunch orientation calls before any KB writes:

```
resolve_repo({ path })           → confirm indexed
plan_turn({ task })              → session orientation
get_repo_outline({ repo })       → file tree, entry points
get_symbol_importance({ repo, top_n: 20 })  → high-value symbols
get_tectonic_map({ repo })       → layer architecture
get_dependency_graph({ repo })   → module relationships
get_hotspots({ repo })           → complexity concentrations
get_dependency_cycles({ repo })  → architectural health
get_untested_symbols({ repo })   → coverage gaps
```

Record in `sessions/log.md`: repo ID, symbol count, file count, key architectural signals.

### Phase 1: KB Project Initialization

1. Run `/project-init` (slug, display name, type)
2. Add type-specific extra directories beyond standard scaffold:
   - `software-lib`: `knowledge/tools/`, `knowledge/agents/`, `knowledge/patterns/`, `knowledge/use-cases/`, `knowledge/integrations/`, `protocols/`, `ingestion/`
   - `standard`: `knowledge/components/`, `knowledge/patterns/`, `protocols/`
   - `research-spike`: `knowledge/findings/`, `knowledge/hypotheses/`
3. Seed scaffold notes from Phase 0 data: `sessions/log.md`, `architecture/index.md`, `ADR-000`, `ADR-001`, `protocols/<slug>-gated-kb-submission-protocol.md`, `planning/roadmap.md`

### Phase 2: Apply the 6-Gate Protocol

Every note passes all 6 gates before `write_note` is called. See full gate definitions below.

### Phase 3: Tiered Ingestion

Work tiers in order:

| Tier | Content | Primary jcodemunch Tools |
|------|---------|--------------------------|
| 1 | Core Architecture (foundation) | `get_tectonic_map`, `get_context_bundle(type_symbols)` |
| 2 | API Surface (tools, endpoints, components) | `get_file_outline` → `get_context_bundle(main_class_ids)` |
| 3 | Patterns and Agents (reusable abstractions) | `get_blast_radius`, `get_ranked_context` |
| 4 | Use Cases (production blueprints) | `get_file_outline` → `get_context_bundle(entry_points)`, `get_symbol_diff` |
| 5 | Operations and Integrations (runbooks, CI/CD) | `get_untested_symbols`, `get_coupling_metrics`, direct file reads |

### Phase 4: Global Truth Promotion

For each pattern reusable beyond this repo:
1. Gate 1 (Bridge Protocol): does it apply generally?
2. Gate 2 dedup: `search_notes("reference/patterns/")` — already exists?
3. If new: write `reference/patterns/<pattern-name>.md`
4. Link from `reference/patterns/` index hub AND from `architecture/index.md`

### Phase 5: Hub Index Updates

After all tiers complete:
1. Update `Index – Agent KB` → add `[[<slug> — README]]` to Projects section
2. Create/update `reference/patterns/Index – Reference Patterns.md`
3. Update relevant capability/domain index with project API surface
4. Mark `planning/roadmap.md` complete with evolution log entry

---

## The 6-Gate Curation Pipeline

Applied to **every note** before `write_note` is called.

### Gate 0 — Ingestion Authorization

**Pass if ALL true**:
- Reveals non-obvious design decision, pattern, or constraint
- Would save a future agent >1 search to recover
- Stable >2 months
- Not trivially derivable by reading code comments or docstrings

**Hard reject**: trivial implementation detail, purely ephemeral state, obvious from reading the source.

### Gate 1 — Classification (Bridge Protocol)

| Classification | Criteria | Target Path |
|---|---|---|
| **Global Truth** | Reusable beyond this repo | `reference/patterns/` |
| **Project-Specific** | Only meaningful to this project | `projects/<slug>/knowledge/` |
| **Hybrid** | Local implementation of global concept | Local note + `[[link]]` to global |

Rule: Promote global truths FIRST. Never bury a reusable pattern inside `projects/`.

### Gate 2 — Pre-Ingest Deduplication

```
1. search_notes(query=<topic>)                         # lexical
2. search_notes(query=<topic>, search_type="hybrid")   # semantic
3. search_notes(tags=["domain/<relevant>"])            # tag-neighborhood
4. build_context(url=nearest_hub, depth=2)             # graph expansion
```

If a suitable note exists: update it (`edit_note`) rather than creating a duplicate.

Also run jcodemunch confirmation before describing a symbol:
```
search_symbols(query=<topic>, detail_level="compact")
```

### Gate 3 — Composition

**Required frontmatter** (all 7 fields):
```yaml
title, created, modified, type, entity_type, status, tags[]
```

**jcodemunch extraction decision tree**:

| Situation | Correct Tool |
|---|---|
| All symbols in a file | `get_file_outline(file_path)` |
| Single target symbol | `search_symbols` → `get_symbol_source(symbol_id)` |
| 2–8 related symbols | `get_context_bundle(symbol_ids[], token_budget, budget_strategy)` |
| Topic-focused context | `get_ranked_context(query, token_budget, scope)` |
| Dependency blast | `get_blast_radius(symbol, depth=2)` |
| Cross-version comparison | `get_symbol_diff(repo_a, repo_b)` |

**Rule**: Never call `get_file_content` without `get_file_outline` first. Never guess symbol IDs.

### Gate 4 — Linking

- [ ] At least one explicit relation (`Implements`, `Related`, `Derived from`, `Extends`, `Used by`)
- [ ] At least one hub/index updated with link to new note
- [ ] Global truths: linked from `reference/patterns/` index AND `architecture/index.md`

### Gate 5 — Verification

- [ ] Re-read note — no internal contradictions
- [ ] All `[[wiki-links]]` resolve (or accepted-unresolved with reason)
- [ ] Status: `evergreen` if complete, `draft` if pending evidence

### Gate 6 — Oversight Report

Session closeout must enumerate:
- Exact note paths created/modified
- Exact tags added/removed
- Exact relation statements and wiki-links added
- Discovery method used at Gate 2
- Unresolved links and placement ambiguities
- What may still be missing or ambiguous

---

## jcodemunch Surgical Extraction Guide

Token savings vs reading files: 80–99.8%. Every code-content extraction uses jcodemunch.

**Setup** (run once per session):
```
resolve_repo({ path })
index_folder({ path, incremental: true })  # only if not indexed
```

**Canonical extraction sequence**:
1. `get_repo_outline` → understand package structure
2. `get_symbol_importance` → identify high-value symbols to document
3. `get_file_outline(target_file)` → enumerate symbol IDs in specific file
4. `get_context_bundle(symbol_ids[], token_budget)` → extract source + imports

**After any manual file edit**:
```
register_edit({ repo, file_paths: [edited_file] })
index_file({ path: /abs/path/to/file })
```

---

## Note Type Reference

| Content | type | entity_type | status |
|---------|------|-------------|--------|
| Architecture overview | `reference` | `reference` | `evergreen` |
| Global reusable pattern | `pattern` | `pattern` | `evergreen` |
| Tool/API reference | `reference` | `reference` | `evergreen` |
| Agent definition | `reference` | `reference` | `evergreen` |
| Production use case | `example` | `example` | `evergreen` |
| ADR | `reference` | `reference` | `evergreen` |
| Runbook | `reference` | `reference` | `evergreen` |
| Protocol/instruction | `instruction` | `instruction` | `evergreen` |
| Session log | `scratch` | `scratch` | `draft` |

---

## Anti-Pattern Registry

| Anti-Pattern | Why Blocked |
|---|---|
| Writing notes about what code comments already say | Zero KB value, noise |
| Burying a reusable pattern in `projects/` folder | Violates Bridge Protocol, prevents reuse |
| Skipping Gate 2 for "obviously new" content | Duplicates accumulate; correctness degrades |
| Notes with no inbound links | Unreachable from any hub; orphaned nodes |
| Notes without Evolution Log (living docs) | History collapse; future agents lose context |
| Guessing symbol IDs without `search_symbols` first | Wrong source, silent drift |
| Reading entire files when a symbol can be extracted | Token waste, context pollution |
| Promoting ephemeral state to `evergreen` | Stale notes mislead future agents |

---

## Parallelization

After Phases 0–2 complete, Tiers 1–5 can be dispatched as parallel sub-agents via `superpowers:dispatching-parallel-agents`. Each tier agent runs the same 6-gate sequence independently and marks its row complete in `planning/roadmap.md`.

---

## Trigger Conditions for Protocol Update

- A new project type requires a different directory scaffold
- Gate 2 repeatedly misses duplicates (search quality regression)
- New jcodemunch tools become available that improve extraction efficiency
- A gate is consistently too strict (blocks valid ingestion) or too loose (allows noise)
- A new note type is needed that doesn't map to existing type/entity_type values

---

## Relations

- Implements: [[Curator Handover Knowledge]]
- Implements: [[Document Curation Metaprompt]]
- Implements: [[Action Taxonomy – Index]]
- Used by: [[agent-toolkit — Gated KB Submission Protocol]]
- Related: [[Agentic Knowledge Base System Overview]]
- Companion skill: `kb:codebase-ingestion` (`/kb-codebase-ingestion`)

---

## Quality Signal Tracking

| Date | Sections Used | Sections Skipped | Gaps Identified |
|------|--------------|-----------------|-----------------|
| 2026-04-18 | All 5 phases, all 6 gates | None | Jupyter notebooks and YAML not AST-indexed — direct Read required |

---

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-04-18 | Initial protocol written | Abstracted from agent-toolkit KB ingestion session (35 notes, 5 tiers, 6 gates, 2 sessions) | User request to create reusable protocol + skill from session execution trace |