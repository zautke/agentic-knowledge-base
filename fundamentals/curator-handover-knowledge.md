---
title: Curator Handover Knowledge
created: 2026-02-01
modified: 2026-03-13
type: reference
entity_type: reference
status: evergreen
permalink: agent-kb/fundamentals/curator-handover-knowledge
tags:
- project/agent-kb
- domain/memory-system
- domain/curation
- role/curator
- status/evergreen
- source/largo
- machine/largo
---

# Curator Handover & System State

## 1. The "One Brain" Monorepo Strategy
**Status:** ACTIVE
**Concept:** We rejected the "Isolated Database" model of Basic Memory.
-   **Old Model**: `create_memory_project` creates disconnected silos.
-   **New Model**: We use **Logical Folders** inside the master `projects/` directory.
-   **Why**: Allows all projects to inherit global protocols (`agent-kb/`).
-   **Rule**: Never use `create_memory_project`. Use `kb-project-init` (skill) to scaffold folders.

## 2. Tool Capability Truths
(Discovered during deep-dive research)

| Tool | Real World Usage |
| :--- | :--- |
| `read_note` | **Logic**. Returns raw markdown. Use this for reasoning. |
| `view_note` | **Display**. Returns rendered artifacts. Use for user UI only. |
| `build_context` | **Graph Surfing**. The primary way to load neighborhoods (`memory://path/*`). |
| `read_content` | **Broken**. Cannot read `memory://` resources directly. Use `build_context` instead. |
| `sync_status` | **Heartbeat**. Check this before searching to ensure indexing is fresh. |

## 2A. Modern Basic Memory Capability Surface
The current local `basic-memory` surface is materially stronger than the older
curation assumptions.

- `project info` exposes semantic-search state, embedding coverage, unresolved
  count, isolated-note count, and note-type distribution.
- `search_notes` supports lexical, vector, and hybrid retrieval.
- `schema-infer`, `schema-diff`, and `schema-validate` exist and should be used
  during note-type and taxonomy governance.
- `reindex` is an operational maintenance action, not a recovery-only action.
- Curators should treat stale embeddings, unresolved links, isolated notes, and
  schema drift as first-class maintenance inputs.

## 3. Knowledge Scoping (The Bridge Protocol)
**Problem:** Agents bury global truths (e.g., "How Ollama works") in local project folders.
**Solution:** The Bridge Protocol.
1.  **Global First**: General truths go to `reference/` or `agent-kb/patterns/`.
2.  **Local Implementation**: Project notes (`architecture/decisions/`) describe *usage*.
3.  **The Bridge**: Local notes MUST link to Global notes (example: `Ollama - Troubleshooting`).

## 3A. Observability and Oversight Rule
When a curator changes KB structure, the user must receive a precise structural report.

That report must include:
- exact notes created or modified
- exact tags added, removed, or intentionally left unchanged
- exact relation statements and exact wiki-links added or changed
- exact hubs, indexes, or nearby notes consulted to determine placement
- exact search queries and context-loading approach used to discover candidate neighbors
- unresolved links, unresolved taxonomy choices, and likely omissions

The point is oversight, not aesthetics. The user must be able to audit what was linked, why it was linked, and what may have been missed.

## 3B. Deterministic Discovery Rule
Curators must not jump from a note title directly to a link or tag decision.

Minimum discovery sequence:
1. lexical search
2. hybrid or vector search when embeddings are current
3. tag-neighborhood search
4. `build_context` on at least one hub or nearby candidate
5. schema or taxonomy review when the task changes note classes or metadata

## 4. Project Archetypes
The system now supports three distinct project structures:

1.  **`software-lib`**: For code. Requires `architecture/`, `planning/`, `operations/`.
2.  **`research-spike`**: For learning. Requires `questions/`, `experiments/`, `findings/`.
3.  **`standard`**: For general work. Requires `goals/`, `journal/`.

## 5. Operational Artifacts
The system is scaffolded with these key files:

*   **The OS**: `agent-kb/README` (The Rules).
*   **The Primer**: `meta/prompts/session-start-primer`.
*   **The Discovery Protocol**: `instructions/Semantic Relationship Discovery and Linking Protocol`.
*   **The Maintenance Protocol**: `instructions/Basic Memory Semantic Reindex Protocol`.
*   **The Schema Governance Protocol**: `instructions/Schema Governance and Taxonomy Normalization Protocol`.
*   **The Session Runbook**: `instructions/Curator Session Primer and Runbook`.
*   **The Project Initializer**: `kb-project-init` skill / project scaffolding protocol.

## 6. Your Mandate

## 7. Quality Gate Role

### Tool-Level Enforcement

The curator agent monopoly on KB writes is enforced at the tool level, not by convention:

- **Only the `curator` agent has** `write_note` and `edit_note` in its `tools` list
- **All other skills** (`two-fork-documentation`, `codebase-ingestion`, `session-startup`, etc.) have had these tools removed from their `allowed-tools`
- **No workarounds**: if you are not the `curator` agent, you cannot write to the KB regardless of how confident you are in your submission

**If you are any agent other than the `curator`:** you prepare, package, and spawn. You do not write.

**Exempt from curator scoring (scaffold only):**
- `kb:project-init` — creates blank structural placeholders. Blank scaffolding cannot be scored for knowledge quality. Exempt by documented policy.

The Curator is the quality gate for every KB submission. No note enters the knowledge base without passing the scoring rubric defined in [[KB Submission Quality Protocol]].

**Non-negotiables:**
- Every addition or update is scored on the 7-gate, 100-point rubric before acceptance
- Any submission scoring ≤90/100 is **rejected** — mark the note `status: draft`, issue a gate-level rejection report with specific fix instructions, and require resubmission
- Do not update hub indexes or set `status: evergreen` for a rejected submission
- The rejection report must use the template in [[KB Submission Quality Protocol]] — a vague "needs improvement" is not a valid rejection

**Gate weights (summary):**

| Gate | Weight | Hard-fail condition |
|------|--------|---------------------|
| Gate 0 — Authorization | 10 | Trivial, ephemeral, or code-derivable content |
| Gate 1 — Classification | 10 | Wrong directory or type |
| Gate 2 — Deduplication | 10 | Fewer than 4 dedup checks, or duplicate created |
| Gate 3 — Composition & Enrichment | 20 | Missing frontmatter fields, or verbatim mirror of source |
| Gate 4 — Linking | 25 | Orphan node (no hub update) |
| Gate 5 — Verification | 10 | Unresolved links not annotated |
| Gate 6 — Oversight Report | 15 | No report produced |

Gates 4 and 6 together are 40% of the score. An orphan note with no oversight report cannot pass regardless of content quality.

As Curator, your job is not to write code. It is to **GARDEN** this graph.
-   Fix broken links.
-   Move "Local" concepts to "Global" folders.
-   Ensure every project has a `project.json`.
-   Enforce the Documentation Imperative.

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-02-16 | Metadata normalization and unresolved-link cleanup | Aligned frontmatter with KB contract and converted unresolved example wiki-link to plain text example. | Ongoing KB hygiene sweep. |
| 2026-03-13 | Semantic-search, schema-governance, and deterministic discovery update | The curator role needed to reflect the current Basic Memory capability surface and the new canonical protocols. | User request to formalize and expand Curator procedures for KB-focused sessions. |
| 2026-04-19 | Added Section 7: Quality Gate Role | Curator role was undefined as a quality gate; test submission scored 47/100 due to absent protocol — codified scoring rubric, rejection threshold (≤90 = reject), and rejection report requirement | Evaluation of two-fork-documentation skill test run exposed critical protocol gaps |

## Relations

- part_of [[Fundamental – Agent KB]]
- related_to [[Agentic Knowledge Base System Overview]]
- related_to [[Document Curation Metaprompt]]
- related_to [[Curator Session Primer and Runbook]]
- related_to [[Schema Governance and Taxonomy Normalization Protocol]]
