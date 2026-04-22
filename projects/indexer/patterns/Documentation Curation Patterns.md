---
title: Documentation Curation Patterns
type: reference
permalink: kb/projects/indexer/patterns/documentation-curation-patterns
tags:
- project/indexer
- domain/patterns
- status/draft
---

# Documentation Curation Patterns

Patterns to use while curating `indexer` knowledge into the canonical project subtree.

## Preferred Patterns
- **Bridge first**: create canonical bridge notes before moving or archiving older notes
- **Implementation truth before aspiration**: current behavior notes should be distinct from target-state synthesis
- **Session evidence matters**: active worktree state and dirty main-branch state should be recorded explicitly
- **Short hubs, rich satellites**: use index notes for navigation and keep detailed evidence in domain notes
- **File-side staging before KB mutation**: when tools are unstable, keep the repo-side audit artifacts authoritative until the KB backend is healthy again

## Anti-Patterns To Avoid
- rewriting older notes in place without preserving provenance
- treating `docs/UNIFIED_ARCHITECTURE.md` as implementation truth
- mixing branch-local work with merged main-branch reality without labeling the difference
- creating duplicate canon without backlinks to the older note network

## Related Notes
- [[KB Backfill Roadmap]]
- [[Documentation Audit and Tech Debt]]
