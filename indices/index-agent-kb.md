---
title: Index – Agent KB
created: 2026-01-01
modified: 2026-03-16
type: index
entity_type: index
status: evergreen
tags:
- project/agent-kb
- status/evergreen
permalink: agent-kb/indices/index-agent-kb
---

# Index – Agent KB

## Root
- [[Fundamental – Agent KB]]

## Core docs

## Observability
- [[Curator Submission Telemetry]] — Append-only log of every Curator scoring event (accepted + rejected); one row per submission with per-gate scores, deduction cause, and advisory; surfaces cumulative patterns and rubric-calibration drift over time

- [[Agentic Knowledge Base System Overview]]
- [[Schema Governance and Taxonomy Normalization Protocol]]
- [[Curator Session Primer and Runbook]]
- [[Machine Profile Collection and Curation Protocol]]
- [[Codebase KB Ingestion Protocol]] — 5-phase blueprint for ingesting any codebase into the KB; 6-gate curation pipeline; jcodemunch extraction guide
- [[KB Submission Quality Protocol]] — 7-gate scoring rubric (100 pts), rejection protocol, and submission checklist; minimum passing score 91; applies to all KB additions and updates
- [[Two-Fork Documentation — Skill Reference]] — filesystem + KB fork skill; curator-gated write workflow; document type → KB type mapping
- [[Curator Agent Deployment Retrospective — 2026-04-19]] — design rationale, deliverables, 10-submission acceptance test (mean 99.2/100), and open gaps from the curator architecture deployment

## Schemas
- [[Instruction Schema]]
- [[Reference Schema]]
- [[Index Schema]]

## Action space
- [[Action Taxonomy – Index]]

## Meta prompts
- [[Meta-Instruction – Add Knowledge README]]
- [[Meta-Instruction – Build Action Taxonomy]]
- [[SessionStart Primer – Agentic Knowledge Base]]

<!-- generated_at: 2026-01-26T22:44:19Z -->

## Projects
- [[multi-agent-platform — README]] — Discovery and planning ledger for a new production multi-agent orchestration platform with Python-first runtime, decoupled dashboard, and full A2A compliance.

- [[Index – Indexer Project]] — Umbrella hub: chatgpt-index + claude-index + evaluation pipeline
- [[Index – blogg Project]] — Next.js tech blog (blogg)
- [[mdeditor]] — React markdown editor
- [[graphrag – System Overview]] — Plugin-driven hybrid RAG pipeline; 19 plugin categories; 5 stages; Textual TUI; JSON-RPC transport
- [[agent-toolkit — README]] — Production Python package: 6 research tools, HybridResearcher agent, 3 use cases; Tavily + LangChain-based

## Registries
- [[Agentic Teams Repository]]
- [[Component Registry]]

## Behavioral Corpora
- [[HOW-AGENTS-GET-FIRED]] — Negative testimonials documenting what agents must never do in the user's employ.
- [[WALL OF EXCELLENCE]] — Affirming counterpart corpus documenting first-person pride notes for execution the user explicitly valued.

## Infrastructure
- [[Machine Profile - largo]] — Canonical host/system + SSH client profile for `largo`
- [[Index - Cloudflare Browser Rendering Operations]] — Hub for the root Cloudflare tunnel, browser-rendered terminal stability model, verification runbook, and dated repair record for iPad-based terminal work on `largo`.
- [[Cloudflare Browser Rendering on largo]] — Canonical entity note for the deployed browser-rendered terminal path, its authoritative config surfaces, and current transport/keepalive decisions.
- [[Machine Profile - adagio]] — Canonical host/system, SSH, WSL, and Docker profile for `adagio`
- [[Operational Session - largo SSH to adagio and tyggr in WSL Mirrored Mode (2026-03-15)]] — Full incident context for mirrored-mode SSH drift, stale local host mapping, Windows-side WSL exposure failure, and related Codex GitHub MCP auth repair.
- [[Local Network Docker Registry on adagio]] — Canonical reference for the LAN-scoped private registry, stack artifacts, endpoints, and image catalog on `adagio`.
- [[Docker Registry Helper Commands on adagio and tyggr]] — Command surface, alias map, and auth precedence for PowerShell and WSL registry workflows.
- [[Operational Session - Local Docker Registry Bootstrap and Registry Auth Repair on adagio and tyggr (2026-03-16)]] — Full audit trail for the registry buildout, shell auth repairs, and repo persistence work.
- [[Operational Audit - adagio LAN Registry Live State and Mirror Gap (2026-03-17)]] — Post-build verification note distinguishing the working private registry from the still-missing mirror/hub and client-trust configuration.
- [[Client Onboarding for adagio LAN Docker Registry]] — Operator-facing trust, login, and direct-vs-mirror onboarding note for LAN clients using `adagio:5443`.
- [[Index - Ollama Local Service Operations]] — Hub for `olsvc.sh` command surface, audit findings, and compatibility/API operations
- [[Ollama Local Service (olsvc.sh) - Operational Session 2026-02-19]] — Session record for field-test artifact ingestion and operational decisions.
- [[Ollama Local Service (olsvc.sh) - Multi-Agent Field Test Plan]] — Reusable multi-agent validation plan with command and coverage inventory.
## Trigger Conditions for Update
- New top-level operational domains are added and require direct root-index discoverability.
- Existing infrastructure hubs split into deeper sub-indices and root links must be rebalanced.
- Repeated retrieval misses indicate current infrastructure links are insufficient.
- New top-level behavioral corpora are added or materially redefined and require direct root-index discoverability.

## Quality Signals
- Useful: Infrastructure hub links now include machine profiles plus Ollama service operations entrypoint.
- Useful: direct links to latest Ollama session + field-test plan reduce lookup hops from root index.
- Useful: direct behavioral-corpus links make both failure patterns and affirming execution standards reachable from the KB root.
- Useful: Cloudflare browser-rendering operations now have a direct root-index path instead of depending on free-text search through machine or session notes.
- Gap: Infrastructure section still has no dedicated network/storage runbook index split.

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-04-21 | Added the Cloudflare browser-rendering operations hub and canonical entity note to Infrastructure. | The KB root index needed direct discoverability for the new browser-rendered terminal domain on `largo` after the tunnel-stability cluster was formalized. | User request to create robust foundational notes around the `cloudflare browser rendering` entity and relate them across the KB. |
| 2026-04-09 | Added `[[HOW-AGENTS-GET-FIRED]]` and `[[WALL OF EXCELLENCE]]` to a new Behavioral Corpora section, plus matching trigger and quality-signal updates. | The KB root index needed direct discoverability for the paired negative and affirming agent-behavior corpora. | User required WALL OF EXCELLENCE to be added to the discoverable index and hardened as an evolving philosophy artifact. |
| 2026-02-16 | Registry-link expansion | Added direct links to active registry hubs for team and component navigation. | Follow-up index hygiene pass after new node creation. |
| 2026-02-18 | Added machine profile infrastructure links | Registered new `largo` profile and `adagio` stub note for infra discoverability. | Curator-Infra machine profiling request. |
| 2026-02-18 | Added Ollama service operations hub link | Registered dedicated `olsvc.sh`/Ollama operational index under Infrastructure for retrieval and session continuity. | Curator session capture for `olsvc.sh` + `/Volumes/DEV/.ollama` audit. |
| 2026-02-18 | Added Infrastructure quality-signal section | Recorded immediate usefulness and remaining index-gap signals after infrastructure index expansion. | Curation-rule compliance pass for quality tracking on touched index note. |
| 2026-02-19 | Added direct links to latest Ollama session and dedicated multi-agent field-test plan note | Ensure top-level discoverability for newly curated operational nodes without requiring hub traversal | User-directed curation ingest of `OLSVC_MULTI_AGENT_FIELD_TEST_PLAN.md` |
| 2026-02-19 | Added infrastructure trigger-conditions section for index maintenance policy | Make update triggers explicit to satisfy living-document curation requirements | Curation protocol hardening pass during infrastructure index update |
| 2026-03-15 | Added direct infrastructure link to the mirrored-mode SSH incident session note for `largo`/`adagio`/`tyggr` | Make the new machine/network incident record discoverable from the root infrastructure index without requiring free-text search | User request to persist and curate the full session context |
| 2026-03-16 | Added direct infrastructure links for the local Docker registry, helper-command note, and registry buildout session note | Make the new machine-local registry cluster directly discoverable from the root infrastructure hub without relying on text search | User request to persist all important aspects of the local registry session |
| 2026-03-16 | Added the machine-profile protocol to Core docs and promoted `adagio` from stub to a full host profile in the infrastructure graph | Future infra curation needed a reusable machine-profile protocol plus a complete `adagio` profile instead of a placeholder | User request to find or create the machine-profile protocol and fill `adagio` with a full system description |
| 2026-03-17 | Added the post-build `adagio` registry live-audit note to Infrastructure | Make the current-state delta discoverable from the root index without requiring cluster traversal or free-text search | User request to audit `adagio`, search the KB, and sync the findings back into the document base |
| 2026-03-17 | Added the `adagio` registry client onboarding note to Infrastructure | Surface the CA-trust and direct-vs-mirror operator note from the root index once the registry follow-up work was completed | User request to complete the three follow-up actions for the `adagio` registry cluster |
| 2026-03-17 | Added `[[Index – assist-indexer Project]]` to Projects section | First session in assist-indexer repo; scaffolded 4 new KB notes for project root, architecture, session log, and project index | /init + /kb-prime session in assist-indexer |
| 2026-03-17 | Added `[[chatgpt-index]]` direct link to Projects section | Implementation repo now has its own KB root note; direct discoverability from root index without hub traversal | Architecture diagram and "Still Missing" gap-fill pass |
| 2026-03-17 | Collapsed indexer project links into single `[[Index – Indexer Project]]` entry; moved all notes from `projects/assist-indexer/` and `projects/chatgpt-index/` to `indexer/`; retagged all 6 notes from `project/assist-indexer`/`project/chatgpt-index` → `project/indexer` | chatgpt-index and claude-index are entities under the umbrella `project/indexer`, not separate top-level projects | User taxonomy correction |
| 2026-03-29 | Added `[[graphrag – System Overview]]` to Projects section | graphrag is a new active project in the KB; root-index discoverability required after 3-layer SOTA research sprint created 11 new notes in `projects/graphrag/` and `reference/rag/` | SOTA research sprint KB curation completion |
