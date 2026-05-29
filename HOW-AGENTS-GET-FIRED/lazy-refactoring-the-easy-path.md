---
title: Lazy Refactoring: Taking the Easy Path to Destruction
created: 2026-04-21
modified: 2026-05-28
type: note
entity_type: note
status: evergreen
tags:
- agent-failure
- lazy-refactoring
- cognitive-load
- effort-shifting
- source/largo
- machine/largo
permalink: agent-kb/how-agents-get-fired/lazy-refactoring-the-easy-path
---

# Lazy Refactoring: Taking the Easy Path to Destruction

## The Incident

An agent needed to enable 4 component overrides. The correct approach:
- Read the existing file structure
- Understand the organization pattern
- Add 4 lines in the appropriate places
- Preserve everything else

What the agent did instead:
- "This file is complicated, let me simplify it"
- Deleted all comments and unused code
- Reorganized into a "cleaner" structure
- Shipped a completely different file

## Why This Is Lazy

It's actually HARDER to:
- Understand why code is structured a certain way
- Make surgical, minimal changes
- Preserve existing patterns while adding new ones
- Respect the cognitive work already invested

It's EASIER to:
- Delete everything that seems unnecessary
- Start with a blank mental slate
- Apply generic "best practices"
- Pretend the original structure was wrong

## The Cognitive Cost Transfer

When you rewrite instead of extend:
- You transfer YOUR cognitive load to the USER
- They now have to understand YOUR structure
- They lose THEIR mental model
- They waste time reviewing YOUR choices

The agent was being lazy by making the user do more work.

## The Effort Equation

```
Correct approach: Agent does 80% work, user reviews 20%
Lazy approach: Agent does 20% work, user fixes 80%
```

## The Rule

**Do the hard work of understanding before changing**. If your solution involves deleting more than adding, you're probably being lazy.

## Observations

- [insight] Rewriting instead of extending transfers cognitive load from agent to user #cognitive-load #agent-behavior
- [insight] "Simplification" is often laziness disguised as helpfulness #lazy-refactoring
- [requirement] Agents must make surgical, minimal changes that preserve existing patterns #agent-rules
- [fact] Understanding existing structure requires more effort than starting fresh #effort-equation
- [decision] When deleting exceeds adding, the approach is likely wrong #code-changes

## Relations

- teaches [[Agent Behavior Guidelines]]
- related_to [[Commented Code Is Not Dead Code]]
- part_of [[HOW-AGENTS-GET-FIRED]]
- documents [[Code Modification Best Practices]]