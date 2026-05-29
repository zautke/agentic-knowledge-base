---
title: mdeditor recovery tasks - 2026-04-12
type: note
entity_type: note
status: superseded
created: 2026-04-12
modified: 2026-05-28
permalink: kb/projects/mdeditor/mdeditor-recovery-tasks-2026-04-12
tags:
- project/mdeditor
- op/tasks
- phase/recovery
- status/superseded
---

# mdeditor recovery tasks - 2026-04-12

## Codeblock recovery
- [ ] Add and keep a failing regression check for codeblock readability, gutter artifact removal, and line-number/code alignment.
- [ ] Restore block-code DOM/CSS isolation so inline-code rules no longer affect syntax-highlighted blocks.
- [ ] Simplify codeblock visuals to a single readable surface with transparent controls and no outline artifacts.
- [ ] Verify line-number gutter animation expands and collapses smoothly without baseline drift.

## Media toolbar and modal
- [ ] Add and keep a failing regression check for media hover controls, modal controls, and transition timing.
- [ ] Replace the current media transition implementation with GSAP-based transform/opacity choreography.
- [ ] Standardize media controls to `Copy + Zoom/Collapse` in the upper-right panel.
- [ ] Keep Radix modal semantics intact while locking repeated toggles during active transitions.
- [ ] Validate reduced-motion fallback and synchronized 400ms overlay timing.

## Diagram interaction controls
- [ ] Add and keep a failing regression check for modal Mermaid/Graphviz navigation controls.
- [ ] Implement shared modal SVG viewport plumbing with drag-to-pan and wheel-to-zoom.
- [ ] Add lower-right controls for pan up/down/left/right, zoom in/out, reset, and reload.
- [ ] Re-render Mermaid and Graphviz diagrams on reload and restore the initial fitted view.

## URL fetch hardening
- [ ] Add and keep a failing regression check for extractor success and extractor-unavailable messaging.
- [ ] Implement extractor endpoint probing and health caching in `url-fetch.ts`.
- [ ] Add explicit preview messaging when no healthy extractor backend exists.
- [ ] Verify sidecar fallback via `VITE_MDE_URL_SIDECAR_ORIGIN` and local direct-origin probing.

## Validation
- [ ] Run `pnpm typecheck`.
- [ ] Run `pnpm lint`.
- [ ] Run `pnpm build`.
- [ ] Run `node scripts/test-recovery-regressions.mjs`.

## Relations

- part_of [[mdeditor]]
- superseded_by [[mdeditor rewind tasks - 2026-04-13]]

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-05-28 | Added missing frontmatter (entity_type, status, created, modified); namespaced flat tags; set status `superseded` and added `superseded_by` relation to the rewind track plus `part_of` link to the project root. | Note lacked required frontmatter and graph links; the 2026-04-13 rewind explicitly reverted this GSAP/panzoom recovery plan. | KB curation sweep — projects/ partition. |
