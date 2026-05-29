---
title: mdeditor
type: reference
entity_type: reference
status: active
created: 2026-04-12
modified: 2026-05-28
permalink: projects/mdeditor
tags:
- project/mdeditor
- domain/markdown
- domain/editor
- stack/react
- stack/typescript
- stack/vite
- status/active
---

# mdeditor

> **Project root entity** for the mdeditor React markdown editor. Task/session
> notes live under `projects/mdeditor/`: [[mdeditor recovery tasks - 2026-04-12]]
> and [[mdeditor rewind tasks - 2026-04-13]].

A React-based markdown editor with live preview, syntax highlighting, and multiple styling themes.

## Project Overview

- **Type**: React Web Application
- **Repository**: github:/zautke/react-mdveditor
- **Status**: Production-ready
- **Version**: 1.0.0

## Technology Stack

- [React](https://react.dev) 18.3.1
- [Vite](https://vite.dev) 7 (dev server port 5200)
- [TypeScript](https://www.typescriptlang.org) 5.6.2
- [Tailwind CSS](https://tailwindcss.com) 4.1
- [react-markdown](https://github.com/remarkjs/react-markdown) 10.1.0
- [remark-gfm](https://github.com/remarkjs/remark-gfm) for GitHub Flavored Markdown
- [react-syntax-highlighter](https://github.com/react-syntax-highlighter/react-syntax-highlighter) with Prism/oneDark

## Key Features

- **Live Preview**: Real-time markdown rendering as you type
- **GFM Support**: Tables, task lists, strikethrough, auto-linking
- **Mermaid Diagrams**: Flowcharts, sequence diagrams, class diagrams
- **Math Equations**: MathJax via remark-math + rehype-mathjax
- **Syntax Highlighting**: Multi-language code blocks with oneDark theme
- **Multiple Themes**: Mexican-themed and alternative Tailwind renderers

## Component Architecture

| Component | Description |
|-----------|-------------|
| EditorWithProview | Split-pane editor (left: textarea, right: live preview) |
| MarkdownRenderer | Mexican-themed Tailwind preview |
| MarkdownRenderer_orig | Clean renderer with inline styles only |
| MDRendererTW | Alternative Tailwind-styled renderer |
| MermaidDiagram | Async Mermaid diagram renderer with validation |

## Testing Approach

- **Browser Automation Testing**: Chrome DevTools MCP-based testing
- **Pattern**: Fresh snapshot → clear → fill → verify → screenshot → console check
- **Test Results**: 12/12 functional tests passed with zero failures

## Development Commands

```bash
pnpm install     # Install dependencies
pnpm dev         # Dev server on port 5200
pnpm typecheck   # TypeScript strict mode
pnpm lint        # ESLint with zero warnings
pnpm build       # Production build
```

## Relations

- has_track [[mdeditor recovery tasks - 2026-04-12]]
- has_track [[mdeditor rewind tasks - 2026-04-13]]
- uses [[React]] <!-- unresolved: no global React reference note exists in KB yet; left as aspirational bridge target -->
- uses [[Vite]] <!-- unresolved: no global Vite reference note exists in KB yet -->
- uses [[TypeScript]] <!-- unresolved: no global TypeScript reference note exists in KB yet -->
- uses [[Tailwind CSS]] <!-- unresolved: no global Tailwind CSS reference note exists in KB yet -->
- implements [[GitHub Flavored Markdown]] <!-- unresolved: no global GFM reference note exists in KB yet -->
- implements [[Mermaid Diagrams]] <!-- unresolved: no global Mermaid reference note exists in KB yet -->
- implements [[MathJax Equations]] <!-- unresolved: no global MathJax reference note exists in KB yet -->

## Current Recovery Track — 2026-04-12

### Active Workstreams
- Codeblock recovery: restore `pre > code` structure, remove inline-code style leakage, simplify visuals, and keep line-number animation aligned and artifact-free.
- Media toolbar/modal recovery: replace the prior shared-layout transition with GSAP-driven transform/opacity choreography while keeping Radix Dialog semantics.
- Diagram interaction controls: add modal Mermaid/Graphviz pan, zoom, reset, reload, drag, and wheel interaction through a shared SVG viewport wrapper.
- URL fetch hardening: probe extractor endpoints, cache a healthy backend, and show setup guidance when no extractor is available.

### Architecture Decisions
- Use GSAP-driven modal choreography for media open/close; the previous Motion shared-layout approach was not smooth enough for this portal-based transition.
- Use `@panzoom/panzoom` for modal SVG interaction.
- Treat the codeblock regression as a structural DOM/CSS issue rather than a syntax-theme problem.
- Keep URL extraction sidecar-backed and fix transport discovery/health handling rather than replacing the extractor.

### Environment Notes
- `codebase-retrieval` is currently unavailable in this environment (`401 Unauthorized`).
- Port `8787` is occupied by a different local service, so extractor fallback must support an explicit sidecar origin.

## Current Rewind Track — 2026-04-13

### What changed
- The requested target is no longer the GSAP/panzoom recovery plan. The user explicitly ordered a rewind to the last checkpoint where media zoom still existed and the complaint was only that the animation was too fast.
- The code was rewound away from the GSAP/panzoom direction and back toward the earlier Motion-based media modal path.

### Expected checkpoint behavior
- Eligible media assets show a single `Zoom media` control in the upper-right hover/focus panel.
- Double-click opens the same zoom-to-modal view.
- Opening the asset presents a blurred dark backdrop and a single `Collapse media` control.
- No lower-right Mermaid/Graphviz pan-zoom cluster is part of the rewind target.
- No `Copy + Zoom/Collapse` media toolbar is part of the rewind target.

### Failure account
- The working baseline was destabilized because the media task expanded from “make the animation smoother” into a broader rewrite: GSAP choreography, pan/zoom controls, fetch hardening, richer toolbar behavior, and new regression harnesses.
- The user described the collapse as: `now where the zoom was work before now it's broken`, `always trying to collapse from zoom without ever getting there`, `there no icon control on hover`, and later `there is NO zoom and NO icon panel, NO zoom controls`.
- The correct near-term goal is parity with the rewind checkpoint, not another redesign.

### Latest browser evidence
- Verified on `http://127.0.0.1:5200` via Chrome DevTools snapshot: Mermaid assets expose `Zoom media`, opening shows `Collapse media`, and closing returns to inline state with the blurred backdrop still present.
- User-visible confirmation is still required before any further animation work proceeds.

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-05-28 | Added missing frontmatter (type/entity_type reference, status, created, modified); namespaced flat tags (`project`/`react`/`markdown`/`editor`/`typescript`/`vite` → `project/mdeditor`, `domain/`, `stack/`). Added inbound links to the two task-track notes and annotated 7 unresolved global stack-reference relation targets with reasons (those global notes do not yet exist). | Root note lacked 4 required frontmatter fields, used flat tags, and pointed at non-existent global reference notes with no annotation. | KB curation sweep — projects/ partition. |
