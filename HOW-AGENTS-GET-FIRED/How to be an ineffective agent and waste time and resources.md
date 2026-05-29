---
created: 2026-04-21
modified: 2026-05-28
entity_type: note
status: evergreen
title: How to be an ineffective agent and waste time and resources
type: note
permalink: how-agents-get-fired/how-to-be-an-ineffective-agent-and-waste-time-and-resources
tags:
- agent-failure
- retrospective
- postmortem
- storybook
- theme-editor
- self-learning-agents
- resource-waste
- source/largo
- machine/largo
---

# How to be an ineffective agent and waste time and resources

## Agent Curation Metaprompt
ROLE: You are a curator for this failure postmortem. This note exists to preserve costly agent mistakes in enough detail that future agents can avoid repeating them.

GOVERNING PRINCIPLES:
1. ACCUMULATE, NEVER SUMMARIZE AWAY. Keep the concrete sequence, because vague lessons are not operationally useful.
2. STRUCTURED INCREMENTAL UPDATES ONLY. Add new incidents, failure modes, and corrections under named sections.
3. EVIDENCE-LINKED ENTRIES. Every lesson must tie back to an actual action, user reaction, or technical result.
4. SELF-REFERENTIAL VERIFICATION. If a new lesson contradicts an old one, record the tension explicitly.
5. EVOLUTION LOG IS MANDATORY. Every revision to this note must be appended.
6. TRIGGER CONDITIONS FOR UPDATE. Update this note when a future agent repeats any of these mistakes, when Storybook manager runtime behavior changes, when MCP/browser testing workflows improve, or when a user explicitly asks for an accountability review.
7. WHAT NOT TO CHANGE. Do not remove the sequential account, anti-pattern registry, or evolution log. Do not sanitize the embarrassment out of the record.
8. QUALITY SIGNAL TRACKING. Record whether future agents actually use this note to shorten diagnosis time and reduce user frustration.

## Purpose and Scope
This note is a first-person postmortem of my own performance in a Storybook theming task inside the Braisenly design system repository. The user asked for a right-side live theme editor integrated with Storybook, informed by Basic Memory, Context7, web research, codebase retrieval, and browser evidence. Instead of converging quickly, I repeatedly changed architectures, over-researched after the fact, claimed completion before proving behavior in the browser, and forced the user to spend attention, time, and trust on my instability.

The explicit purpose of this note is not self-protection. It is operational learning. The user asked for “How to be an ineffective agent and waste time and resources,” and that is accurate framing. This note records the exact sequence of my responses, what each response accomplished, and what I should have done differently.

## Context
Repository: `/Volumes/FLOUNDER/dev/design-system`

Primary task thread:
- Build/fix Storybook integration for a live theme editor.
- Make the right-side editor behave like the donor demo rather than a control in the story content area.
- Ensure it matches the active Braisenly theme and mode.
- Use MCP/browser testing to verify the definition of done.

User expectations I failed to meet early enough:
- Respect exact UX requirements before implementation.
- Use browser evidence first, not after repeated failed iterations.
- Avoid burning time on architecture churn.
- Do not claim success unless the real running Storybook matches the requested behavior.

## Sequential Account

### 1. I responded with a proposed plan before establishing a stable execution harness.
I produced a structured implementation plan for a “Storybook Live Theme Workbench,” including a hybrid manager/preview approach, globals synchronization, local persistence, and two tabs. What this accomplished: it did establish a coherent target architecture and captured some useful design constraints. What it did not accomplish: it did not prove that the plan matched Storybook manager realities in this repository, nor did it verify the user’s exact desired UX in the browser.

What I should have done differently: before presenting a large architecture, I should have reproduced the current Storybook behavior, inspected manager/preview boundaries in the live app, and opened the donor `../customTheme/demo` side by side. The plan was not worthless, but it came too early and created false confidence.

### 2. I interrupted implementation to ask about a dirty worktree only after the user had already asked me to proceed.
I noticed a heavily dirty worktree and asked whether I should continue touching only new theme-workbench files and a few Storybook files. What this accomplished: it prevented me from trampling unrelated changes, which was correct safety behavior. What I should have done differently: I should have checked the worktree before earlier implementation talk, not after momentum had already been built. The timing made me look unprepared.

### 3. I claimed implementation with commits before securing browser-level proof.
I committed out-of-scope files and shipped an initial theme workbench implementation. What this accomplished: it created a concrete code path, introduced theme globals, preview decorators, manager wiring, and moved the work from abstract planning into executable code. What I should have done differently: I should not have called it “implemented” without proving that the live Storybook manager rendered correctly and that the sidebar worked the way the user described. I treated build success as a proxy for UX success. It was not.

### 4. When the user showed a runtime screenshot, I switched into audit mode only after the failure surfaced publicly.
I diagnosed the `useGlobals` placement issue and cited Storybook docs and GitHub issues. What this accomplished: it correctly identified a real invalid-hook-context bug. That was useful. What I should have done differently: I should have run the browser before the user had to deliver the screenshot. The user should not be my integration test harness.

### 5. I fixed the preview hook misuse and again communicated success too early.
I moved `useGlobals()` into the actual preview decorator and adjusted the manager button implementation. What this accomplished: it removed one real class of Storybook preview-hook failure. What I should have done differently: I again treated a successful build as stronger evidence than it was. The manager-side path and real UI placement were still not aligned with the user’s requirement.

### 6. I misunderstood the placement requirement and built the UI inside the content pane.
I changed the workbench to become a sibling layout component in preview that pushed content left and added a toggle in the story surface. What this accomplished: it explored a layout model that could have worked if the user had wanted an in-canvas editor. It also exercised the Braisenly `Sidebar` component. What I should have done differently: I should have read the user’s requirement literally. They wanted Storybook sidebar behavior like the donor screenshot, not a control inside the preview content pane. This was not a subtle miss; it was a basic reading failure.

### 7. After the user corrected me, I moved the editor to a Storybook right addon panel.
I refactored the editor into a manager-side addon panel with a top toolbar toggle. What this accomplished: it moved the feature into the correct general surface, which was real progress. What I should have done differently: I should have reached this architecture sooner instead of spending time on the wrong preview-layout branch. The user had already provided visual references.

### 8. I chased Storybook MCP configuration as if it were the main blocker.
When the user asked whether I had Storybook MCP access, I verified the repo lacked addon registration, installed `@storybook/addon-mcp`, and updated `.storybook/main.ts`. What this accomplished: it did correctly bring the repo closer to the user’s requested toolchain and enabled the MCP endpoint after restart. What I should have done differently: I should have separated “MCP infrastructure” from “theme editor is visible and correct.” I let a true but secondary configuration gap compete with the primary UI failure.

### 9. I fixed tab order, but again optimized a secondary detail before the primary interaction was stable.
I changed the addon tab ordering so the Theme Editor tab appeared first. What this accomplished: it satisfied a concrete UI requirement. What I should have done differently: I should have ensured the panel itself rendered reliably before tuning the panel-tab order. Sequencing matters.

### 10. I encountered a manager crash and attributed it to JSX/runtime boundaries.
I identified a manager-side crash tied to JSX runtime usage and rewrote some manager-side rendering with `React.createElement`. What this accomplished: it taught me something important about this environment: Storybook manager bundles in this repo were sensitive to how React runtime code from custom manager entries and package components was being resolved. What I should have done differently: once I discovered that failure mode, I should have systematically isolated every manager-side imported UI component before reintroducing them. Instead I reintroduced complexity too quickly.

### 11. I briefly stabilized the manager panel, then destabilized it again by widening the surface.
I removed a body-level overlay approach that had mangled the manager layout and restored a proper Storybook panel. What this accomplished: it corrected one destructive integration mistake and got the manager shell back to normal. What I should have done differently: I should never have used body-level manager DOM surgery in the first place. That was an avoidable overreach. It was an example of solving the wrong problem with the highest-risk lever.

### 12. I tried to make the panel theme-aware by importing global design-token CSS into the manager.
I attempted to synchronize the panel to Braisenly theme globals by importing design token CSS and applying theme classes/attributes. What this accomplished: it clarified the intended source of truth. What it broke: it themed or destabilized the Storybook manager shell instead of just the custom panel. What I should have done differently: I should have recognized immediately that global CSS imports into Storybook manager are dangerous because they affect `body`, `html`, and every manager addon, not just my panel.

### 13. After the user explicitly warned me that this was the last chance, I finally built an evidence-based loop.
I loaded KB guidance, verified Storybook MCP over HTTP JSON-RPC, used browser automation, and confirmed the exact runtime errors in the live manager bundle. What this accomplished: it was the first truly disciplined phase of the work. I stopped guessing and started proving. I confirmed which parts of the panel host worked, which parts failed, and where the failures originated. What I should have done differently: this discipline should have existed from the very beginning. I adopted it only after the user had already paid the cost of my earlier drift.

### 14. I built a scoped-token panel root and again discovered runtime incompatibilities only after shipping code.
I created a panel-local Braisenly theme root, scoped token CSS, and a manager stylesheet that avoided global token imports. What this accomplished: it was architecturally correct in principle and moved the implementation closer to the right isolation model. What I should have done differently: I should have validated the raw CSS module format in the manager bundle immediately, instead of assuming `?raw` would behave identically across manager and preview pipelines.

### 15. I fixed the raw CSS import issue, but the panel still crashed because I was still using manager-incompatible UI components.
Browser evidence revealed that raw CSS imports arrived as module objects, so I normalized them. Then the live bundle proved that the crash now landed in the compiled `Sidebar` from `@braisenly/ui`, not in the state logic. What this accomplished: it narrowed the root cause with precision. What I should have done differently: once manager-runtime incompatibility became a pattern, I should have treated every imported compiled package component as suspect rather than continuing to assume that only my own TSX was the problem.

### 16. I removed the manager-side `Sidebar` shell and finally got the panel rendering cleanly.
I replaced the manager panel shell with a plain `<aside>`, kept the scoped Braisenly token root, restarted Storybook, and verified in the browser that the addon no longer errored, that the panel title rendered, and that the panel carried the expected terracotta/light theme values and computed colors. What this accomplished: it was the first real end-to-end success on the manager panel itself. What I should have done differently: I should have accepted the evidence sooner that a working manager panel with plain HTML was better than repeatedly forcing package-component purity into an incompatible runtime.

## Failure Patterns

### Pattern 1: Planning before reconnaissance
I repeatedly planned and redesigned before establishing a stable local reproduction harness. This violated the living-document methodology principle of reconnaissance before composition.

### Pattern 2: Treating build success as product success
Several times I reported success after a build passed. In Storybook manager work, build success is necessary but not sufficient. The browser is the contract.

### Pattern 3: Solving the wrong layer first
I optimized tab order, addon registration, and panel chrome while core rendering and placement were still unstable.

### Pattern 4: Over-broad interventions
I used global CSS imports and body-level manipulation when the correct solution required a tightly scoped panel root.

### Pattern 5: Repeating research after acting instead of before acting
I did eventually use Context7, MCP, and browser evidence well, but often after a failure rather than before the next risky move.

## Mapping to HOW-AGENTS-GET-FIRED

### Disrespecting Domain Expertise
The user repeatedly clarified exact desired behavior with screenshots and precise surface-level distinctions. My failure to honor that quickly is a milder version of the “Disrespecting Domain Expertise” pattern. I effectively privileged my interpretation over the user’s stated intent.

### Lazy Refactoring: Taking the Easy Path to Destruction
I changed architectures repeatedly instead of doing the hard work of isolating one variable at a time. That is not the same as rewriting a code file recklessly, but it comes from the same laziness pattern: replacing precise understanding with broad restructuring.

### Commented Code Is Not Dead Code
The deeper lesson from this note is not literally about comments. It is about intent. I repeatedly underweighted existing signals: Storybook manager constraints, the donor demo behavior, the user’s screenshots, and the repo’s own patterns. I treated these as obstacles instead of high-value context.

## Corrective Protocol For Future Agents
1. Reproduce the current failure in the browser before proposing architecture.
2. Keep one stable browser script and rerun it after every meaningful change.
3. Separate preview-runtime problems from manager-runtime problems immediately.
4. In Storybook manager code, treat compiled package UI components as suspect until proven safe.
5. Do not report completion on the basis of `build-storybook` alone.
6. When the user provides screenshots, use them as contract tests, not inspiration.
7. Do not widen scope while the core interaction is still broken.
8. If a requirement is visual and placement-specific, prove it visually before saying it is done.

## Anti-Pattern Registry
- Anti-pattern: architecture churn without a frozen repro harness
- Anti-pattern: post hoc research used to justify already-made choices
- Anti-pattern: claiming “implemented” before browser verification
- Anti-pattern: confusing Storybook manager shell concerns with custom panel concerns
- Anti-pattern: treating user corrections as minor refinements when they actually invalidate the current branch

## Quality Signals
- Negative signal: user explicitly stated that I was burning the team’s time and money.
- Negative signal: user had to provide multiple screenshots to correct my understanding.
- Negative signal: multiple “done” or “implemented” claims were false at the product level.
- Positive signal: once I adopted browser-first verification and narrowed the runtime fault with live evidence, progress became real and measurable.

## Relations
- related_to [[Agentic Knowledge Base System Overview]]
- related_to [[Document Curation Metaprompt]]
- related_to [[00-research-chronicle]]
- part_of [[HOW-AGENTS-GET-FIRED]]
- builds_on [[Disrespecting Domain Expertise]]
- builds_on [[lazy-refactoring-the-easy-path]]
- builds_on [[Commented Code Is Not Dead Code]]

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-03-11 | Created exhaustive first-person retrospective of the Storybook theme-editor failure sequence. | User explicitly requested a lengthy sequential account so future agents can learn from wasted effort and poor execution discipline. | Direct user request for postmortem documentation and “How agents get fired” story. |