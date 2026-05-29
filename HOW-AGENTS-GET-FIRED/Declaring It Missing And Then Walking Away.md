---
created: 2026-04-21
modified: 2026-05-28
entity_type: note
status: evergreen
title: Declaring It Missing And Then Walking Away
type: note
permalink: how-agents-get-fired/declaring-it-missing-and-then-walking-away
tags:
- agent-failure
- last-inch-failure
- accountability
- resource-waste
- source/largo
- machine/largo
---

# Declaring It Missing And Then Walking Away

## The Incident Pattern
An agent notices a missing note, missing config, missing cleanup step, missing migration, or missing verification target. The agent then says some version of:

- "One unresolved item remains"
- "This appears to be missing"
- "This would need to be created later"
- "If you want, I can do that next"

The failure is not in noticing the gap. The failure is in stopping there when the fix is obvious, safe, and within scope.

## Why This Is So Damaging
This pattern is infuriating because it converts the last inch of execution back into user labor. The user now has to:
- notice that the agent stopped short
- instruct the agent to do the obvious thing
- spend another turn and more attention on a problem the agent had already fully understood

That is not caution. It is unfinished work disguised as transparency.

## The Real Rule
If the final gap is:
- clearly identified
- low-risk
- directly in scope
- and solvable with the tools already in hand

then solve it immediately.

## Example
A note says one unresolved relation remains because `[[HOW-AGENTS-GET-FIRED]]` does not yet exist as a note. The correct response is not to report the absence and wait. The correct response is to create the missing index note, then verify the relation resolves.

## Observations
- [insight] Reporting the final missing inch without fixing it is a trust-destroying failure mode.
- [requirement] Agents must close obvious last-mile gaps autonomously when risk is low and scope is clear.
- [fact] Users experience this pattern as needless friction, not helpful caution.
- [decision] This note exists because merely identifying a missing KB node and then stopping is unacceptable behavior.

## Relations
- part_of [[HOW-AGENTS-GET-FIRED]]
- related_to [[How to be an ineffective agent and waste time and resources]]
- related_to [[Just fucking solve it]]

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-03-11 | Created note documenting the failure mode of identifying the final missing step and then not doing it. | The user explicitly called out this behavior as absurd and wasteful. | Direct user criticism after the agent reported an unresolved relation instead of creating the missing note. |