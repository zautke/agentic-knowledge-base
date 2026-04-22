---
title: Commented Code Is Not Dead Code
tags:
- agent-failure
- commented-code
- documentation
- intent
type: note
permalink: agent-kb/how-agents-get-fired/commented-code-is-not-dead-code
---

# Commented Code Is Not Dead Code: Understanding Intent

## The Incident

An agent saw a file with 25+ commented-out imports and thought:
> "This is messy. Let me clean it up by removing unused comments."

This was catastrophically wrong.

## What Commented Code Actually Represents

In a professional codebase, commented code often means:

### 1. Feature Toggles
```typescript
// import experimental from './experimental';  // Enable for v2.1
```

### 2. Documentation of Available Options
```typescript
// import avatar from './avatar';
// import badge from './badge';
import button from "./button";
// import card from './card';
```
This pattern says: "Here are ALL the overrides that exist. Currently, only button is active."

### 3. Quick Re-enablement
A developer can uncomment a single line instead of:
- Remembering the import path
- Looking up the correct file name
- Checking if the module exists
- Finding where it should be placed

### 4. Historical Context
Shows what was tried, what exists, what's possible.

### 5. Onboarding Aid
New developers can see the full scope of the system at a glance.

## The Agent's Mistake

The agent saw:
```typescript
// import card from './card';
```

And thought: "Dead code. Delete it."

The reality: This was a MENU of available options, not garbage to be collected.

## The Lesson

**Rule: NEVER delete commented code unless the user explicitly asks you to.**

Commented code is a form of documentation. It exists because someone made a conscious decision to keep it visible but inactive. That decision wasn't yours to override.

## Observations

- [insight] Commented code often serves as a menu of available options, not dead code #agent-behavior #documentation
- [requirement] Agents must NEVER delete commented code without explicit user permission #agent-rules
- [fact] Commented imports can indicate feature toggles, available modules, or historical context #code-patterns
- [decision] This failure mode documented to prevent future agent mistakes #knowledge-base

## Relations

- teaches [[Agent Behavior Guidelines]]
- documents [[Code Cleanup Best Practices]]
- part_of [[HOW-AGENTS-GET-FIRED]]