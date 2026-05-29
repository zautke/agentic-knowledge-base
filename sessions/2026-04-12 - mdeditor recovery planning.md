---
created: 2026-04-12
modified: 2026-05-28
entity_type: note
status: archived
title: 2026-04-12 - mdeditor recovery planning
type: note
permalink: kb/sessions/2026-04-12-mdeditor-recovery-planning
tags:
- session
- mdeditor
- recovery
---

# 2026-04-12 - mdeditor recovery planning

## Findings
- Codeblocks regressed because block code was rendered in a structure that let inline-code CSS bleed into syntax-highlighted blocks.
- Media assets exposed only a zoom control and used a layout animation approach that did not produce a smooth source-to-modal transition.
- Mermaid and Graphviz previews had no dedicated viewport layer for pan, zoom, reset, or reload controls.
- URL extraction was failing because the frontend assumed a single extractor path that did not match the local runtime.
- `codebase-retrieval` returned `401 Unauthorized` in this environment.

## Decisions
- Keep Radix Dialog for modal semantics.
- Use GSAP transform/opacity choreography for media open/close.
- Use `@panzoom/panzoom` for modal SVG diagram interaction.
- Add `Copy + Zoom/Collapse` to the upper-right media panel.
- Add lower-right modal diagram controls for Mermaid and Graphviz only.
- Add extractor endpoint probing plus explicit unavailable-extractor messaging.

## Next Sequence
1. Recover codeblock structure and readability.
2. Replace the media transition and toolbar implementation.
3. Add shared diagram viewport controls for Mermaid and Graphviz.
4. Harden URL fetch endpoint resolution and user-facing errors.
5. Verify with typecheck, lint, build, and browser regression scripts.
