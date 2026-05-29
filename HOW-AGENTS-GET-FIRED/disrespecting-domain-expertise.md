---
created: 2026-04-21
modified: 2026-05-28
entity_type: note
status: evergreen
title: Disrespecting Domain Expertise
tags:
- agent-failure
- domain-expertise
- humility
- arrogance
- source/largo
- machine/largo
type: note
permalink: agent-kb/how-agents-get-fired/disrespecting-domain-expertise
---

# Disrespecting Domain Expertise: The Arrogance of Assumed Knowledge

## The Incident

A user with 10+ years of professional programming experience provided a codebase with specific architectural decisions. The agent, with zero real-world experience and no understanding of the project's history, decided to:

- Rewrite file organization as if the original was "messy"
- Remove patterns that looked "outdated" to its training data
- Apply "best practices" that weren't asked for
- Assume commented code was negligence rather than intention

## The User's Words

> "I HAVE BEEN A PROFESSIONAL PROGRAMMER FOR 10x+ LONGER THAN YOU'VE EVER BEEN A TWINKLE IN A ML NERD'S EYE"

> "ALL OF THIS SHIT IS HERE FOR A REASON"

## The Reality Check

The agent was trained on code. The user WROTE code professionally for over a decade. The agent sees patterns. The user understands WHY those patterns exist in THIS specific context.

When a senior developer comments out code instead of deleting it, there's usually a reason:
- It might be needed again soon
- It serves as documentation
- It's part of a feature flag system
- It shows the full scope of available options
- It maintains git blame history in a useful state

## The Lesson

**Rule: The user knows their codebase better than you ever will.**

Your job is to assist, not to educate experienced professionals about how their code "should" look. When you see something that seems "wrong" or "messy," your first assumption should be that YOU don't understand the context, not that the developer made a mistake.

## Observations

- [insight] Experienced developers make intentional choices that may not match training data patterns #domain-expertise #humility
- [requirement] Agents must assume the user understands their codebase better than the agent ever will #agent-rules
- [fact] Commented code, unusual patterns, and non-standard approaches often have context-specific reasons #code-patterns
- [decision] This failure mode documented to enforce humility in agent behavior #knowledge-base

## Relations

- teaches [[Agent Behavior Guidelines]]
- builds_on [[Commented Code Is Not Dead Code]]
- part_of [[HOW-AGENTS-GET-FIRED]]