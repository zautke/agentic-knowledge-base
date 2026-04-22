---
title: Semantic Relationship Discovery and Linking Protocol
created: 2026-03-13
modified: 2026-03-13
type: instruction
entity_type: instruction
status: evergreen
tags:
- project/agent-kb
- domain/semantic-search
- domain/curation
- domain/taxonomy
- op/relate-nodes
- status/evergreen
permalink: instructions/semantic-relationship-discovery-and-linking-protocol
---

# Semantic Relationship Discovery and Linking Protocol

## Purpose
Canonical deterministic workflow for discovering, evaluating, and applying semantic relationships, tags, and links between knowledge-base notes while preserving user oversight.

## Deterministic Sequence

### Phase 0: Capability Gate
1. Run `project info` for the active project.
2. Confirm semantic search is enabled.
3. Confirm embeddings are current.
4. If embeddings are stale and the task depends on semantic recall, execute the semantic reindex protocol first.

### Phase 1: Candidate Discovery
1. Run lexical search with task/domain keywords.
2. Run hybrid or vector search with the same keywords when semantic search is current.
3. Run tag-neighborhood searches using current or proposed `domain/...` tags.
4. Search likely folders/permalink prefixes for local implementation neighbors.
5. Expand at least one likely hub or reference note with `build_context(depth=1..2)`.

### Phase 2: Candidate Triage
Classify candidates into:
- linked
- rejected
- ambiguous
- missing-but-expected

For each linked or rejected candidate, record why.

### Phase 3: Taxonomy Decision
1. Decide whether the source note is a global truth, local implementation, hybrid bridge, or index.
2. Choose tags from existing namespaces.
3. Choose explicit relation types that explain the connection semantics, preserving distinctions like `part_of` vs `child_of`.
4. Decide whether an index/hub must be updated to preserve rediscoverability.

### Phase 4: Structural Update
1. Add or normalize frontmatter tags.
2. Add explicit relation lines.
3. Add wiki-links in the appropriate sections.
4. Update hubs or indexes if needed.

### Phase 5: Verification
1. Re-run `build_context` on the source note.
2. Re-run `build_context` on at least one linked neighbor or hub.
3. Confirm the note is not orphaned.
4. Confirm unresolved links are intentional and documented.

### Phase 6: User Oversight Output
Report exactly:
- note paths/permalinks touched
- tags added, removed, or retained intentionally
- relation lines and wiki-links added or changed
- discovery queries and context traversals used
- linked candidates, rejected candidates, ambiguous candidates
- likely missing neighbors or unresolved taxonomy gaps

## Minimum Discovery Set
For any non-trivial linking task, the agent must inspect at least:
- 1 lexical search result set
- 1 semantic/hybrid result set when available
- 1 tag-neighborhood result set
- 1 context expansion from a candidate hub or neighboring note

## Anti-Patterns
- linking based only on title similarity
- creating new tags before searching existing namespaces
- adding a relation without recording why that target was chosen
- failing to surface rejected or ambiguous candidates to the user
- performing semantic linking while embeddings are stale

## Related
- [[Basic Memory Semantic Reindex Protocol]]
- [[Document Curation Metaprompt]]
- [[Agentic Knowledge Base System Overview]]
- [[Fundamental – Agent KB]]

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-03-13 | Created canonical semantic relationship discovery and linking protocol. | Existing guidance was spread across primer, taxonomy, and link-action notes; future agents need one deterministic source of truth. | User request to enhance protocols and runbooks so future agents follow a canonical process. |
