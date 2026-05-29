---
title: Basic Memory Semantic Reindex Protocol
type: instruction
permalink: instructions/basic-memory-semantic-reindex-protocol
created: 2026-03-13
modified: 2026-03-13
entity_type: instruction
status: evergreen
tags:
- project/agent-kb
- domain/basic-memory
- domain/semantic-search
- domain/maintenance
- op/hygiene
- status/evergreen
- source/largo
- machine/largo
---

# Basic Memory Semantic Reindex Protocol

## Purpose
Reusable maintenance protocol for rebuilding Basic Memory full-text and semantic indexes, measuring wall-clock latency, and preserving an append-only performance log for future agents.

## When To Run
- `project info` reports `Reindex recommended`.
- Embeddings indexed count is below total entity count.
- Search results appear stale, sparse, or semantically weak.
- Basic Memory is upgraded and vector-search features need to be activated for existing notes.
- A large batch of KB notes was added, renamed, or substantially rewritten.
- Foundational protocols or taxonomy runbooks were materially updated and agents will depend on semantic retrieval to discover them.

## Preconditions
- Confirm available disk space before running.
- Confirm semantic search is enabled for the target project.
- Expect the first run after upgrade or cache miss to take longer because the embedding model may need to download.

## Canonical Commands

### Full reindex for one project
```bash
/usr/bin/time -p basic-memory reindex -p kb
```

### Embeddings only
```bash
/usr/bin/time -p basic-memory reindex -p kb --embeddings
```

### Search index only
```bash
/usr/bin/time -p basic-memory reindex -p kb --search
```

## Required Verification Steps
1. Run `basic-memory project info kb` before the reindex and note the embedding status.
2. Run the timed reindex command.
3. Run `basic-memory project info kb` again.
4. Record wall-clock duration from the `real` line.
5. Update `Last Run Duration` and append a new row to `Latency Log`.
6. In the user-facing closeout, report whether the run included model download/setup overhead.
7. Report pre-run and post-run indexed entity counts exactly.

## Last Run Duration
- Last run timestamp: `2026-03-14 03:15 MDT`
- Scope: full reindex for project `kb`
- Wall-clock duration: `36.86s`
- Overhead notes: warm-cache run after relation-taxonomy normalization in control docs, parser-safe relation-template cleanup, and frontmatter tag cleanup in touched meta surfaces.
- Post-run status: `Up to date`, `113/113` entities embedded, `2728` chunks indexed.

## Future Agent Instructions
- Always measure with `/usr/bin/time -p` so the `real` duration is captured consistently.
- Always record whether the run was cold-cache or warm-cache.
- Do not overwrite historical timings; append a new row to the log.
- If the run fails, still append a row with the failure reason and stop point.
- If embeddings remain below total entities after a successful run, escalate and inspect semantic-search configuration or provider availability.
- If foundational discovery/linking protocols changed, mention that this run refreshes semantic discoverability for those instructions.

## User Oversight Output
- exact command run
- exact project targeted
- exact pre-run embedding status
- exact post-run embedding status
- exact wall-clock duration
- whether the run was cold-cache or warm-cache
- whether the run included dependency/model download overhead

## Latency Log
| Date | Project | Command | Duration (real) | Cache State | Outcome | Notes |
|------|---------|---------|-----------------|-------------|---------|-------|
| 2026-03-13 12:51 MDT | kb | `basic-memory reindex -p kb` | `213.51s` | cold | success | Included initial FastEmbed model download; post-run status `104/104` embedded, `1920` chunks, `Up to date`. |
| 2026-03-13 13:04 MDT | kb | `basic-memory reindex -p kb` | `20.77s` | warm | success | Reindexed after protocol/runbook updates; post-run status `106/106` embedded, `2169` chunks, `Up to date`. |
| 2026-03-13 14:27 MDT | kb | `basic-memory reindex -p kb` | `19.17s` | warm | success | Reindexed after Curator protocol hardening and `dot-agents` project creation; post-run status `109/109` embedded, `2363` chunks, `Up to date`. |
| 2026-03-13 14:38 MDT | kb | `basic-memory reindex -p kb` | `12.17s` | warm | success | Reindexed after creating schema notes and schema-governance navigation links; post-run status `112/112` embedded, `2464` chunks, `Up to date`. |
| 2026-03-13 14:47 MDT | kb | `basic-memory reindex -p kb` | `22.43s` | warm | success | Reindexed after relation-vocabulary normalization across reference, session, project, and action notes; post-run status `112/112` embedded, `2464` chunks, `Up to date`. |
| 2026-03-14 03:15 MDT | kb | `basic-memory reindex -p kb` | `36.86s` | warm | success | Reindexed after preserving `part_of` vs `child_of`, normalizing relation-taxonomy guidance, cleaning parser-visible relation examples, and fixing malformed tag lists in touched meta docs; post-run status `113/113` embedded, `2728` chunks, `Up to date`. |
## Related
- [[Fundamental – Agent KB]]
- [[Agentic Knowledge Base System Overview]]
- [[Document Curation Metaprompt]]
- [[Semantic Relationship Discovery and Linking Protocol]]
- [[Index – MCP Gateway Operations]]

## Evidence
- Pre-run: `basic-memory project info kb` showed semantic search enabled but `0/83` indexed and `Reindex recommended`.
- Run command: `/usr/bin/time -p basic-memory reindex -p kb`
- Post-run: `basic-memory project info kb` showed `104/104` indexed, `1920` chunks, status `Up to date`.
- Warm rerun command: `/usr/bin/time -p basic-memory reindex -p kb`
- Warm rerun post-run: `basic-memory project info kb` showed `106/106` indexed, `2169` chunks, status `Up to date`.
- Latest rerun command: `/usr/bin/time -p basic-memory reindex -p kb`
- Latest rerun post-run: `basic-memory project info kb` showed `109/109` indexed, `2363` chunks, status `Up to date`.
- Schema-sync rerun command: `/usr/bin/time -p basic-memory reindex -p kb`
- Schema-sync rerun post-run: `basic-memory project info kb` showed `112/112` indexed, `2464` chunks, status `Up to date`.
- Current rerun command: `/usr/bin/time -p basic-memory reindex -p kb`
- Current rerun post-run: `basic-memory project info kb` showed `113/113` indexed, `2728` chunks, status `Up to date`.

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-03-13 | Created semantic reindex protocol with latency logging requirements. | Future agents need a repeatable maintenance action with measurable performance history. | User request to run reindexing, time it, and persist a reusable protocol. |
| 2026-03-13 | Updated last-run baseline and appended warm-cache timing sample. | Future agents need both cold-cache and warm-cache expectations for planning and diagnostics. | Protocol/runbook updates required a fresh semantic reindex. |
| 2026-03-13 | Appended a second warm-cache sample after Curator and `dot-agents` updates. | The latency history should reflect later-stage steady-state performance after KB growth and control-plane hardening. | User request to execute the protocol improvements and preserve action latency history. |
| 2026-03-13 | Appended a schema-sync warm-cache sample after creating executable schema notes. | The latency history should include the cost of making schema governance discoverable and validated. | User request to proceed with real schema creation and validation. |
| 2026-03-13 | Appended a relation-normalization warm-cache sample after canonicalizing `related_to` variants. | The latency history should capture normalization work that materially changes semantic retrieval. | User request to continue normalization. |
| 2026-03-14 | Appended a control-surface normalization warm-cache sample after preserving distinct hierarchy semantics and cleaning parser-visible examples. | Future agents need the latest baseline after relation-taxonomy guidance and meta-doc normalization changed the graph surface. | User request to continue normalization while keeping `part_of` and `child_of` distinct. |
