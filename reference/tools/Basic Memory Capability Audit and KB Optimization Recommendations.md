---
title: Basic Memory Capability Audit and KB Optimization Recommendations
type: reference
permalink: reference/tools/basic-memory-capability-audit-and-kb-optimization-recommendations
created: 2026-03-13
modified: 2026-03-13
entity_type: reference
status: evergreen
tags:
- project/agent-kb
- domain/basic-memory
- domain/semantic-search
- domain/schema
- domain/taxonomy
- domain/curation
- status/evergreen
---

# Basic Memory Capability Audit and KB Optimization Recommendations

## Purpose
Audit the current capability surface of modern Basic Memory, compare it to the live `kb` project state, and recommend schema, taxonomy, discovery, and curation improvements that increase agentic experiential growth, retrieval breadth, and graph visibility.

## Upstream Capability Surface

### Core Product Capabilities Confirmed
- Basic Memory `0.20.2` supports local-first knowledge management with markdown-backed entities, observations, and relations.
- The CLI exposes operational commands for `status`, `reindex`, `doctor`, `mcp`, `format`, `update`, `import`, `cloud`, `tool`, `project`, and `schema`.
- The tool/MCP layer exposes note creation and editing, `read_note`, `search_notes`, `build_context`, and `recent_activity`.
- `search_notes` supports multiple retrieval modes: lexical text/title/permalink plus `vector`/`semantic` and `hybrid` search modes.
- Semantic retrieval exposes quantitative scoring and a configurable `min_similarity` threshold.
- `project info` exposes embedding health, embedding provider/model, indexed counts, unresolved counts, isolated-note counts, and note-type distributions.
- Schema tooling now exists via `schema validate`, `schema infer`, and `schema diff`.
- Semantic indexing is backed by vector chunking and embeddings, with `reindex` supporting full, search-only, or embeddings-only rebuilds.

### Retrieval Details That Matter Operationally
- Hybrid/vector retrieval is only trustworthy when embeddings are current.
- Quantitative semantic scoring exists in the search path, but it is query-to-result scoring rather than a first-class note-A vs note-B comparison API.
- `build_context` remains essential because semantic ranking alone does not expose graph topology or relation semantics.
- Tags, explicit relation blocks, and hubs still matter even with semantic search enabled because they constrain and explain graph structure.

## Live `kb` Project State On 2026-03-13
- Basic Memory version in use during audit: `0.20.2`.
- Semantic search is enabled for project `kb`.
- Embedding provider/model: `fastembed` / `bge-small-en-v1.5`.
- After warm reindex, project status was `Up to date` with `106/106` entities embedded and `2169` chunks indexed.
- Current graph still shows `47` unresolved relations and `2` isolated notes.

## What The KB Is Not Yet Leveraging Well Enough

### 1. Schema System Is Present But Underused
- `schema diff` for `instruction`, `reference`, and `index` found no stored schemas for those dominant note types.
- This means the KB currently has schema tooling available but is not using it to constrain drift in its most important note classes.
- The project is still relying more on prose conventions than executable schema contracts.

### 2. Note-Type Drift Exists
Observed type values include:
- `reference`, `instruction`, `note`, `index`
- but also `meta_prompt`, `meta-instruction`, `technical-documentation`, `knowledge_base`, `example`, `design-decisions`, `context`, `api-reference`, `algorithm-reference`
- and one clearly malformed legacy value: `concept | decision | project | reference | meeting | person`

This weakens filtering, schema inference quality, and retrieval precision.

### 3. Relationship Semantics Are Still Inconsistent
- Inferred schema/drift output showed overlapping relation names in live use, including `related_to`, `relates_to`, `uses`, `implements`, `child_of`, `part_of`, `links_to`, and one-off relation phrases leaking into relation blocks.
- This reduces graph coherence and makes automated reasoning about relation meaning less reliable.

### 4. Semantic Search Is Now Available, But Process Has Historically Been Lexical/Manual
- The KB had strong manual curation patterns before embeddings were available.
- That is useful, but it means there is likely under-linking of semantically adjacent notes that were not lexically obvious.
- The new deterministic protocols should push agents to treat hybrid/vector retrieval as a required candidate-discovery step rather than an optional extra.

## Highest-Leverage Improvements

### A. Create Real Schemas For Core Note Types
Start with executable schemas for:
- `instruction`
- `reference`
- `index`
- `fundamental`
- `meta_prompt`

Benefits:
- makes drift visible earlier
- gives agents stronger creation/update constraints
- improves consistency of tags, relations, and required sections
- turns KB discipline into enforceable structure rather than memory

### B. Normalize Type Taxonomy Aggressively
Move toward a small canonical set plus explicit metadata dimensions.

Recommended pattern:
- keep `type` limited to a compact canonical set
- move specialization into tags or metadata such as `subtype`, `domain`, `role`, `artifact_class`

Example:
- instead of separate top-level note types like `api-reference` or `algorithm-reference`, keep `type: reference` and add tags like `artifact/api`, `artifact/algorithm`, or metadata such as `reference_kind: api`.

### C. Normalize Relation Vocabulary
Pick a canonical relation vocabulary and convert aliases or prose-like relation lines to that vocabulary.

Recommended minimal set for this KB:
- `implements`
- `uses`
- `related_to`
- `part_of`
- `specializes`
- `documents`
- `indexed_by` or `contains` for hubs, but choose one primary convention

The goal is not maximum expressiveness. The goal is stable semantics for retrieval and reasoning.

### D. Add A Required Semantic Discovery Layer To Every Curation Task
For any non-trivial note creation or linking task, require:
1. lexical keyword search
2. hybrid/vector search
3. tag-neighborhood search
4. context expansion from at least one candidate hub
5. explicit reporting of linked, rejected, ambiguous, and missing candidates

This is the main change that will maximize breadth and visibility.

### E. Reduce Unresolved And Isolated Notes As A Standing Maintenance Target
The unresolved count and isolated count should be treated as first-class health metrics.

Recommended standing targets:
- unresolved relations trend should move downward over time
- isolated notes should be reviewed at every maintenance pass
- every new note should have at least one hub/index path and one semantically adjacent note path where appropriate

## Recommended Schema/TAXONOMY Refactor Strategy

### Phase 1: Canonicalize Structural Fields
- Standardize `type`
- Standardize `entity_type`
- Standardize `status`
- Standardize relation section names
- Standardize tag namespace prefixes

### Phase 2: Introduce Core Schemas
- Infer initial schemas from high-quality notes
- manually tighten them
- validate the project against them
- use drift reports as recurring maintenance input

### Phase 3: Add Specialized Metadata Without Exploding Type Count
Introduce metadata fields such as:
- `reference_kind`
- `instruction_kind`
- `scope` (`global`, `local`, `bridge`, `index`)
- `domain_cluster`
- `maintenance_tier`

This preserves expressiveness while keeping the top-level taxonomy compact.

### Phase 4: Build Better Hubs
For every major domain cluster, maintain:
- one global reference/fundamental note
- one protocol/instruction note
- one index/hub note
- explicit bridge links to operational/local runbooks

This is the right model for clusters like MCP, cloudflared, supergateway, and Basic Memory itself.

## How To Best Leverage Newer Functionality For Agentic Growth

### Use `project info` As A Mandatory Session Gate
Agents should begin by learning whether semantic retrieval is currently trustworthy, not assuming it.

### Use Hybrid Search For Discovery, Not Final Authority
Hybrid/vector search is best used to surface candidate neighbors. Final linking should still be justified by note purpose, taxonomy, and relation semantics.

### Use Schema Diff As A Curation Trigger
Schema drift output should become a maintenance input, not an occasional diagnostic curiosity.

### Treat Reindex Latency As Operational Knowledge
The KB now has a measured cold-cache and warm-cache baseline for semantic indexing. This should continue for future maintenance operations so agents can plan around real cost, not guesswork.

### Create More Audit Notes Like This One
Capability audits are useful because they convert product evolution into KB operating guidance. Future upgrades should produce an updated capability audit and a reindex entry.

## What We Could Be Doing Better Right Now
- define and store schemas for dominant note types
- normalize outlier `type` values
- normalize relation vocabulary
- add a shared tag/metadata registry note for canonical namespaces and allowed specializations
- create maintenance dashboards or index notes for unresolved relations and isolated notes
- apply semantic-neighborhood linking to older clusters that were created before embeddings existed
- add periodic audit cadence for drift, orphaning, and weak-domain coverage

## Suggested Next Actions
1. Create canonical schemas for `instruction`, `reference`, and `index`.
2. Run a type-normalization pass for outlier note types.
3. Run a relation-normalization pass for aliases and prose-like relation statements.
4. Create a domain/tag registry note with allowed namespaces and specialization metadata.
5. Run semantic-neighborhood audits on high-value clusters, starting with MCP/cloudflared/basic-memory maintenance knowledge.

## Related
- [[Basic Memory Semantic Reindex Protocol]]
- [[Semantic Relationship Discovery and Linking Protocol]]
- [[Document Curation Metaprompt]]
- [[Agentic Knowledge Base System Overview]]
- [[Fundamental – Agent KB]]

## Evidence
- Local CLI inspection on 2026-03-13.
- `project info kb` before and after reindex.
- Local source inspection of `search_notes`, schema, project info, and semantic retrieval code.
- Schema inference performed for `instruction`, `reference`, and `index`.

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-03-13 | Created capability audit and KB optimization reference note. | The project needed a grounded audit of current Basic Memory functionality plus concrete optimization recommendations for schema, taxonomy, and agentic growth. | User request to explain current capabilities, audit the KB, and persist the findings. |

## Sources
- Local Basic Memory `0.20.2` CLI and installed source inspection
- Basic Memory docs and repo pages reviewed during the audit
