---
title: Migration Plan - agent-kb to agentic-kb Consolidation
type: note
entity_type: note
permalink: operations/migration-plan-agent-kb-to-agentic-kb-consolidation
created: 2026-04-21
modified: 2026-05-28
status: active
tags:
- project/agent-kb
- domain/migration
- domain/operations
- op/consolidation
- status/active
---

# Migration Plan: agent-kb → agentic-kb Consolidation

## Overview

| Attribute | Value |
|-----------|-------|
| **Source Project** | `agent-kb` (proto-project) |
| **Target Project** | `agentic-kb` (authoritative) |
| **Status** | 📋 Planned |
| **Created** | 2026-01-01 |

---

## Current State

### agentic-kb (Target - Authoritative)
- **Entities**: 196
- **Observations**: 797
- **Relations**: 310
- **Files**: 159 (51 directories, 108 files)
- **Role**: Primary knowledge base for all agentic operations

### agent-kb (Source - Proto)
- **Entities**: 20
- **Observations**: 103
- **Relations**: 68
- **Files**: 17 (11 directories, 6 files)
- **Role**: Was incorrectly set as default; contains newer notes to migrate

---

## Files in agent-kb to Migrate

| File | Path | Action |
|------|------|--------|
| shadcn Tailwind Design System Adoption | `sessions/2026-01-01 - shadcn...` | **MIGRATE** - New session note |
| Agentic Teams Repository | `teams/Agentic Teams Repository.md` | **SKIP** - Duplicate (agentic-kb has authoritative version) |
| README | `meta/README.md` | **REVIEW** - May have unique content |
| design-system-deployment-protocol | `protocols/...` | **MIGRATE** - New protocol |
| design-system-workflow | `operations/...` | **MIGRATE** - New workflow |
| mdeditor | `projects/mdeditor.md` | **REVIEW** - Check if exists in agentic-kb |

---

## Migration Steps

### Phase 1: Audit (Before Migration)
1. [ ] Read each agent-kb note to understand content
2. [ ] Check for corresponding notes in agentic-kb
3. [ ] Identify unique vs duplicate content
4. [ ] Document merge strategy for overlaps

### Phase 2: Migration Execution
1. [ ] For each unique note in agent-kb:
   - Read full content from agent-kb
   - Write to agentic-kb in same folder structure
   - Verify note exists in agentic-kb
2. [ ] For duplicates with newer content:
   - Compare timestamps and content
   - Merge or replace as appropriate
3. [ ] Update any cross-references to point to agentic-kb paths

### Phase 3: Verification
1. [ ] List all agentic-kb directories to confirm migration
2. [ ] Search for migrated note titles
3. [ ] Verify relations are intact
4. [ ] Run `/list-teams` to confirm team registry works

### Phase 4: Cleanup
1. [ ] Document agent-kb as deprecated
2. [ ] Consider deleting agent-kb project (preserves files on disk)
3. [ ] Restart Basic Memory to apply default project change

---

## Migration Commands

```javascript
// For each note to migrate:

// 1. Read from source
switch_project("agent-kb")
read_note("path/to/note")

// 2. Write to target
switch_project("agentic-kb") 
write_note({
  title: "Note Title",
  folder: "target/folder",
  content: "<content from step 1>"
})

// 3. Verify
read_note("target/folder/note-title")
```

---

## Post-Migration Verification

```javascript
// Verify default project
get_current_project()  // Should show agentic-kb

// Verify team registry
list_directory("teams/registered", depth=2)

// Test /list-teams command
// Should show RES-001 and any other registered teams
```

---

## Rollback Plan

If migration fails:
1. agent-kb files remain untouched on disk
2. Can re-add agent-kb project if needed
3. No destructive operations until Phase 4

---

## Observations

- [decision] agentic-kb is authoritative; agent-kb was proto-project
- [fact] Default project changed from agent-kb to agentic-kb (requires restart)
- [requirement] All new notes should go to agentic-kb going forward
- [lesson] Multiple projects can cause confusion; consolidate when possible

## Relations

- part_of [[Index – Agent KB]]

## Unresolved Relations (forward-references, annotated 2026-05-28)

These targets describe the `agentic-kb` consolidation destination and do not resolve inside this `agent-kb` repo. Left as documented forward-references; resolve them after the migration lands and the destination notes exist.

- governs `[[teams/Agentic Teams Repository]]` — target lives in the agentic-kb destination, not yet present here
- affects `[[protocols/Team Registration Protocol]]` — target lives in the agentic-kb destination, not yet present here
- documents `[[meta/agentic-kb Overview]]` — target lives in the agentic-kb destination, not yet present here

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-04-21 | Migration plan captured (git import date) | Record the agent-kb → agentic-kb consolidation plan | Project consolidation decision |
| 2026-05-28 | Completed frontmatter to 7-field KB contract (added created/modified/status/entity_type, normalized bare tags to `domain/`/`op/` taxonomy); linked into root `[[Index – Agent KB]]`; demoted 3 forward-reference relations to a documented unresolved block | Note was an orphan with bare tags and 3 dangling forward-links to the not-yet-existing destination KB | KB curation hygiene sweep (operations/reference partition) |