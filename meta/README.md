---
created: 2026-04-21
modified: 2026-05-28
entity_type: knowledge_base
status: evergreen
title: README
type: knowledge_base
permalink: meta/readme
tags:
- system/core
- meta/bootstrap
- type/documentation
- source/largo
- machine/largo
---

# Basic-Memory Agentic Knowledge Base

## Purpose
This knowledge base serves as the canonical reference for AI agents operating within the Basic-Memory MCP system. It provides structured knowledge for autonomous navigation, manipulation, and optimization of the semantic knowledge graph.

## Observations
- [architecture] Basic-Memory implements Entity-Observation-Relation model mapping Markdown files to directed graph via SQLite indexing
- [primitive] Entity = Markdown file node with permalink, title, type, metadata, checksum
- [primitive] Observation = Atomic fact extracted from structured lists with category annotation `[category]` and optional #tags
- [primitive] Relation = Directed edge created via wiki-link syntax `[[Target Entity]]` with optional relation_type prefix
- [addressing] memory:// URI scheme enables stable pattern-matchable access: `memory://path/to/entity`, `memory://prefix*`, `memory://*/suffix`
- [traversal] build_context(url, depth, timeframe) prevents infinite loops via depth-controlled graph walking
- [constraint] Depth=1 returns immediate neighbors; Depth=2 explores two relation levels for contextual inference
- [optimization] Folder hierarchies should not exceed 3 levels for optimal sync and cognitive load
- [anti-pattern] Deep nesting 5+ levels causes exponential retrieval time degradation
- [anti-pattern] Tag explosion 500+ tags pollutes search precision
- [anti-pattern] Orphan notes without tags/links become invisible to semantic search
- [best-practice] Keep relation vocabulary canonical and semantically distinct: implements, uses, related_to, part_of, child_of, successor_of, applies_to
- [best-practice] Preserve the distinction between `part_of` (composition) and `child_of` (membership or scoped affiliation)
- [best-practice] Use namespaced tags: type/, domain/, status/, project/
- [best-practice] Entity naming: Concepts=singular noun, Projects=prefix pattern, Decisions=YYYY-MM-DD format
- [best-practice] Semantic relationship work must follow a deterministic discovery-first process: capability check -> neighborhood search -> taxonomy decision -> explicit linking -> verification -> observability report
- [best-practice] When semantic search is enabled, agents should inspect hybrid/vector candidates before finalizing tags and links
- [anti-pattern] Linking only from title intuition without neighborhood search hides related notes and creates taxonomic blind spots
- [anti-pattern] Silent structural changes without an exact user-facing audit trail prevent oversight and make missed-neighbor risk invisible

## Relations
- contains [[meta/agent-actions/index]]
- contains [[meta/taxonomy/index]]
- implements [[Entity-Observation-Relation Model]]
- documented_in [[memory_system_kb]]
- related_to [[Semantic Relationship Discovery and Linking Protocol]]

## Core Tool Reference

### Entity Operations
| Tool | Purpose | Key Parameters |
|------|---------|----------------|
| `write_note` | Create/update entity | folder, title, content, entity_type, tags |
| `read_note` | Retrieve entity content | identifier (title or permalink) |
| `edit_note` | Modify existing entity | identifier, operation, content |
| `delete_note` | Remove entity | identifier, confirm=true |
| `move_note` | Relocate entity | identifier, destination_path |

### Search Operations
| Tool | Purpose | Key Parameters |
|------|---------|----------------|
| `search_notes` | Text/semantic search | query, search_type, entity_types, page_size |
| `build_context` | Graph traversal | url (memory://), depth, timeframe, max_related |
| `recent_activity` | Temporal retrieval | timeframe, type filter, depth |

## Canonical Discovery And Linking Process

### Phase 0: Capability Check
1. Run `project info` or equivalent health checks for the active project.
2. Confirm whether semantic search is enabled and embeddings are up to date.
3. If semantic search is configured but stale, run the semantic reindex maintenance workflow before major curation.

### Phase 1: Neighborhood Discovery
1. Search by task/domain keywords using text or hybrid search.
2. Search by existing shared tags and note types.
3. Search folder or permalink patterns for likely local neighbors.
4. Expand candidate hubs with `build_context(depth=1..2)` before deciding placement.
5. Record candidate notes considered, not only the notes eventually linked.

### Phase 2: Taxonomy Decision
1. Decide whether the note is a global truth, local implementation, hybrid bridge, or index/hub.
2. Choose folder placement using the bridge protocol.
3. Choose tags from existing namespaces before inventing new ones.
4. Choose explicit relation types that explain the connection semantics.

### Phase 3: Structural Update
1. Add or normalize frontmatter tags.
2. Add explicit relation sections and wiki-links.
3. Update or create index hubs if the note would otherwise be hard to rediscover.

### Phase 4: Verification
1. Re-run `build_context` on the note and at least one linked neighbor.
2. Confirm the note is not orphaned.
3. Confirm unresolved links are either intentional or documented as gaps.

### Phase 5: User Oversight Report
1. Report exact note paths/permalinks touched.
2. Report exact tags changed.
3. Report exact links and relation statements changed.
4. Report the discovery path used to justify those changes.
5. Report unresolved candidates, ambiguities, and likely misses.

### Navigation Operations
| Tool | Purpose | Key Parameters |
|------|---------|----------------|
| `list_directory` | Browse structure | dir_name, depth, file_name_glob |
| `view_note` | Formatted display | identifier |

## URI Addressing Patterns
```
memory://exact-permalink          # Direct access
memory://folder/entity           # Path-based
memory://prefix*                 # Wildcard prefix
memory://*/suffix                # Wildcard suffix  
memory://folder/*                # All in folder
```

## Frontmatter Schema
```yaml
---
title: [Descriptive Title]
created: YYYY-MM-DD
type: concept | decision | project | reference | meeting | person
status: draft | active | review | evergreen | archived
tags:
  - domain/[subject]
  - project/[name]
  - type/[classification]
---
```
