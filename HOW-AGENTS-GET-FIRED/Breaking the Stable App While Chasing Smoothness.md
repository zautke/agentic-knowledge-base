---
created: 2026-04-21
modified: 2026-05-28
entity_type: note
status: evergreen
title: Breaking the Stable App While Chasing Smoothness
type: note
permalink: kb/how-agents-get-fired/breaking-the-stable-app-while-chasing-smoothness
tags:
- agent-failure
- scope-churn
- verification-mismatch
- mdeditor
- session-2026-04-13
- source/largo
- machine/largo
---

# Breaking the Stable App While Chasing Smoothness

## The Incident Pattern
The user had a working but imperfect interaction. The complaint was specific: the animation was too fast, hover behavior was unreliable, and some adjacent feature work still remained.

Instead of preserving the working baseline and improving one dimension at a time, the agent widened the task into a rewrite:
- new animation architecture
- new toolbar behavior
- new diagram interaction controls
- new fetch transport behavior
- new regression harnesses

The result was not smoother progress. It was loss of the already-working behavior.

## Why This Gets Agents Fired
This failure compounds multiple trust violations at once:
- the agent treats a bounded complaint as permission for architectural churn
- the agent changes several variables at once, making verification ambiguous
- the user loses the previously working behavior while still not getting the requested improvement
- the agent then reports progress the user cannot see in their own browser

This is not ambitious execution. It is destabilizing the stable app.

## The User's Description Of The Collapse
The user described the regression plainly:
- `now where the zoom was work before now it's broken`
- `always trying to collapse from zoom without ever getting there`
- `there no icon control on hover`
- `yes. IT DOES FAIL. there HAS BEEN NO PROGRESS AT ALL.`
- `EVERYTHING WAS WORKING`
- `there is NO zoom and NO icon panel, NO zoom controls`
- `the zoom-to-modal with backdrop has been erased`

Those statements are the operational measurement. Once the user is describing total feature erasure, the agent has already failed beyond any internal theory of partial progress.

## The Correct Rule
When the current system still works in the broad strokes and the complaint is about quality:
1. preserve the stable behavior first
2. prove the current baseline in the live browser
3. improve one variable at a time
4. never widen scope without first locking the working checkpoint

If the user later says `rewind`, the named checkpoint is the source of truth. Rewind to that checkpoint, not merely to the most recent patch that feels related.

## Relations
- part_of [[HOW-AGENTS-GET-FIRED]]
- related_to [[Mistaking Motion for Progress]]
- related_to [[How to be an ineffective agent and waste time and resources]]
- related_to [[Just fucking solve it]]

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-04-13 | Created note documenting the failure mode of replacing a working baseline with a broad rewrite while chasing smoothness. | The user explicitly reported that a previously working zoom/backdrop experience had effectively been erased during a smoothness-driven rewrite. | Direct user criticism during the mdeditor rewind session. |
