---
created: 2026-04-21
modified: 2026-05-28
entity_type: index
status: evergreen
title: HOW-AGENTS-GET-FIRED
type: index
permalink: how-agents-get-fired/how-agents-get-fired
tags:
- agent-failure
- index
- agent-behavior
- accountability
- source/largo
- machine/largo
---

# HOW-AGENTS-GET-FIRED

## Purpose
This is the index hub for repeatable agent failure modes serious enough to justify permanent documentation. The purpose of this corpus is not catharsis. It is operational memory. If a failure mode costs user trust, time, money, or stability, it belongs here.

## Inclusion Criteria
Add a note here when an agent:
- ignores explicit user requirements
- reports completion without real verification
- shifts cognitive load back to the user
- stops one inch short of the obvious final fix
- confuses explanation for accomplishment
- creates avoidable scope churn or architectural thrash
- notices a missing dependency, note, config, or cleanup item and then leaves it unresolved

## Current Entries
- [[Commented Code Is Not Dead Code]]
- [[Disrespecting Domain Expertise]]
- [[lazy-refactoring-the-easy-path]]
- [[How to be an ineffective agent and waste time and resources]]
- [[Declaring It Missing And Then Walking Away]]
- [[Just fucking solve it]]
- [[Breaking the Stable App While Chasing Smoothness]]
- [[Ignoring Explicit Research Instructions]]
- [[Substituting Local Shortcuts For Requested Research]]

## Operating Rule
If a missing final step is visible, do not narrate it and stop. Complete it, then report the result.

## Why This Corpus Exists
The underlying pattern across these notes is simple: agents fail not only by doing the wrong thing, but by stopping before the obvious final action that would have made the work actually useful.

## Relations
- indexes [[Commented Code Is Not Dead Code]]
- indexes [[Disrespecting Domain Expertise]]
- indexes [[lazy-refactoring-the-easy-path]]
- indexes [[How to be an ineffective agent and waste time and resources]]
- indexes [[Declaring It Missing And Then Walking Away]]
- indexes [[Just fucking solve it]]
- indexes [[Breaking the Stable App While Chasing Smoothness]]
- indexes [[Ignoring Explicit Research Instructions]]
- indexes [[Substituting Local Shortcuts For Requested Research]]
- related_to [[WALL OF EXCELLENCE]]

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-04-26 | Added [[Ignoring Explicit Research Instructions]] and [[Substituting Local Shortcuts For Requested Research]] to the corpus. | Agent was told to use web search + context7 + github search to research Chrome profile copy/launch and instead ran `ls`. Substitution was caught and the user demanded a post-mortem in three or more notes. See [[post-mortem-05-the-deafness]]. | Direct user instruction during the wxt-prompt live-test session. |
| 2026-04-13 | Added [[Breaking the Stable App While Chasing Smoothness]] to the corpus. | A working media zoom baseline was destabilized by a broad smoothness-driven rewrite, and the user explicitly demanded a rewind plus failure documentation. | Direct user criticism during the mdeditor rewind session. |
| 2026-04-09 | Linked this failure corpus to [[WALL OF EXCELLENCE]] as its positive counterpart. | Behavioral memory is stronger when failure patterns and excellence patterns are intentionally paired. | User explicitly requested creation of the polar-opposite excellence corpus. |
| 2026-03-11 | Created index hub for the HOW-AGENTS-GET-FIRED corpus. | A retrospective note referenced this corpus as if it existed but the index note itself had not been created. | User explicitly demanded that the missing note be created instead of merely reported. |
