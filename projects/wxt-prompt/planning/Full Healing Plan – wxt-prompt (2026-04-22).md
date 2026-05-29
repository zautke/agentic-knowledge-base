---
title: Full Healing Plan – wxt-prompt (2026-04-22)
type: reference
permalink: agent-kb/projects/wxt-prompt/planning/full-healing-plan-wxt-prompt-2026-04-22
status: draft
entity_type: reference
tags:
- project/wxt-prompt
- domain/planning
- domain/documentation
- status/draft
---

# Full Healing Plan – wxt-prompt (2026-04-22)

## Purpose
Canonical KB log of the full proposed healing plan for the `wxt-prompt` project.

## Source
- Assistant proposed plan dated 2026-04-22.
- Canonical filesystem copy: `/Volumes/FLOUNDER/dev/wxt-prompt/docs/superpowers/plans/2026-04-22-wxt-prompt-full-healing-plan.md`

## Verbatim Plan

# wxt-prompt Full Healing Plan

## Summary

Execute a full healing pass across both:

- `agent-kb` operational notes for `wxt-prompt`
- repo-local docs in `/Volumes/FLOUNDER/dev/wxt-prompt`

This pass is not a narrow note update. It is a domain repair sweep intended to leave the `wxt-prompt` project discoverable, current, bridge-linked to the right global notes, and free of known stale planning guidance.

## Required Healing Operations

### 1. Repair the repo-local docs first

Update these repo files so the filesystem truth matches the approved direction:

- `PLANNING.md`
  - preserve old content
  - append the approved 2026-04-21 plan verbatim as the new authoritative direction
  - mark older planning assumptions superseded
- `TASKS.md`
  - replace the current active/next-task framing with the 6-track program:
    1. runtime contracts and reducer
    2. transcript/session store extraction
    3. LiteLLM gateway transport and model catalog
    4. MCP browser capability boundary
    5. `useAgentLoop` compatibility adapter
    6. cutover, dead-code removal, regression hardening
- `docs/architecture/agent-loop-ripout-wire-map.md`
  - keep factual wire map
  - add status section noting changed guidance
- create one new repo-local program doc if needed:
  - `docs/architecture/litellm-gateway-runtime-refactor-program.md`
  - canonical repo-local home for the approved plan if `PLANNING.md` becomes too mixed

### 2. Heal the `agent-kb/projects/wxt-prompt` cluster

Update existing notes:

- `wxt-prompt – Project Overview`
  - move from stale runtime-replacement framing to the approved major refactor framing
  - keep `status: draft` only if unresolved questions remain after the update; otherwise promote to `evergreen`
  - remove stale implication that provider-specific runtime remains the architectural baseline
- `projects/wxt-prompt/planning/Agent Loop Replacement Planning – wxt-prompt`
  - preserve prior content
  - append a `Superseded by 2026-04-21 Program` section
  - copy the approved plan verbatim into the note
  - explicitly mark prior assumptions as superseded in the evolution log
- `projects/wxt-prompt/architecture/Agent Loop Rip-Out Wire Map – wxt-prompt`
  - keep factual wire inventory
  - add a forward-looking note that current replacement guidance has changed
- `projects/wxt-prompt/Index – wxt-prompt Project`
  - add links to all new workstream notes
  - add the global runtime/gateway/capability references

Create the missing project-local entities:

- `projects/wxt-prompt/planning/LiteLLM Gateway Runtime Refactor Program – wxt-prompt`
- `projects/wxt-prompt/planning/Runtime Contracts and Reducer – wxt-prompt`
- `projects/wxt-prompt/planning/Transcript and Session Store Extraction – wxt-prompt`
- `projects/wxt-prompt/planning/LiteLLM Gateway Transport and Model Catalog – wxt-prompt`
- `projects/wxt-prompt/planning/MCP Browser Capability Boundary – wxt-prompt`
- `projects/wxt-prompt/planning/useAgentLoop Compatibility Adapter – wxt-prompt`
- `projects/wxt-prompt/planning/Cutover Dead-Code Removal and Regression Hardening – wxt-prompt`
- `projects/wxt-prompt/operations/Operational Audit – wxt-prompt KB Domain Drift (2026-04-21)`

### 3. Bridge the project correctly to the global domain space

Update global notes only where the current project decision materially sharpens them:

- `LiteLLM in Agentic Systems`
  - append browser-extension guidance:
    - best fit is proxy/gateway HTTP boundary
    - not an in-browser SDK substrate
    - direct local Ollama may remain as explicit local mode
- `MCP Capability Boundaries in Agentic Systems`
  - append browser-extension example:
    - page context and browser tools belong at the capability boundary
    - runtime still owns thread state, turn reduction, and transcript persistence
- `Index – Agent Runtime and Capability Stack`
  - add `wxt-prompt` program hub as a project-local example if appropriate

Do not create a project-specific contribution protocol for `wxt-prompt`. Use the existing global curator/submission/linking protocols as the governing protocol layer.

### 4. Normalize known KB drift exposed by this audit

Within the touched `wxt-prompt` notes:

- normalize relation vocabulary to the current canonical forms
- remove duplicated `relates_to` / `related_to` style duplication where it exists in note bodies
- ensure every new or updated note has:
  - valid frontmatter
  - explicit relations
  - inbound hub link
  - evolution-log entry

Outside the `wxt-prompt` cluster:

- do not run a broad KB-wide normalization sweep in this execution
- do record the schema drift findings in the operational audit note:
  - `reference` drift: `relates_to`, `part_of`
  - `index` drift: `relates_to`, `links_to`

### 5. Add the strategic fork in `agentic-kb`

Create exactly one strategic note:

- `research/wxt-prompt-runtime-refactor-research`

Contents:

- LiteLLM proxy/gateway findings
- `curator-adk` / ADK-style runtime findings
- explicit statement that `agent-kb/projects/wxt-prompt` is the operational source of truth
- links back to the project program hub and the global runtime stack notes

Do not create duplicate project planning notes in `agentic-kb`.

## Test and Verification

After healing, verify both filesystem and KB state.

Filesystem checks:

- `git diff -- PLANNING.md TASKS.md docs/architecture/agent-loop-ripout-wire-map.md docs/architecture/litellm-gateway-runtime-refactor-program.md`
- confirm the approved plan text appears verbatim in the chosen canonical repo doc
- confirm no repo doc still presents the old LiteLLM/MCP/UI assumptions as current

KB checks:

- `search_notes(project="agent-kb", query="\"wxt-prompt\"")` returns the expanded cluster
- `build_context(project="agent-kb", url="memory://projects/wxt-prompt/*")` shows a coherent project neighborhood without duplicated stale planning dominance
- `search_notes(project="agentic-kb", query="\"wxt-prompt\"")` returns exactly one strategic research note
- `Index – wxt-prompt Project` links to all new planning/workstream notes
- touched notes use canonical relation vocabulary
- touched notes are not orphaned

## Assumptions

- “Copy the plan verbatim” means the approved 2026-04-21 refactor plan from the previous turn is inserted without paraphrasing into the authoritative repo doc and the authoritative project KB planning note.
- “Perform ALL healing operations necessary” for this run means:
  - full repair of the `wxt-prompt` KB domain
  - alignment of repo docs with that repaired domain
  - targeted global-note bridge updates
  - not a repo-wide or KB-wide global hygiene sweep beyond what this project materially depends on
- The current strongest confirmed defects are stale planning, missing scope entities, weak bridge coverage, and local relation-vocabulary drift.

## Relations
- part_of [[Index – wxt-prompt Project]]
- related_to [[Agent Loop Replacement Planning – wxt-prompt]]
- related_to [[wxt-prompt – Project Overview]]

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-04-22 | Created canonical KB copy of the full healing plan. | The full proposed plan needed a durable, discoverable project-local note rather than remaining only in chat history. | User request to confirm the entire plan is logged on the filesystem and in the KB. |