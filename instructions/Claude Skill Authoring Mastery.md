---
title: Claude Skill Authoring Mastery
type: instruction
permalink: agent-kb/instructions/claude-skill-authoring-mastery
status: active
tags:
- claude-code
- skills
- skill-authoring
- token-efficiency
- cso
- agentskills
---

# Claude Skill Authoring Mastery

## Core Principle

**"Only add context Claude doesn't already have."**

A skill is NOT a tutorial. It is a compact reference guide containing only the non-obvious, non-general information an agent needs to operate a proven technique or tool.

**Context engineering rule:** "Find the smallest possible set of high-signal tokens that maximize the likelihood of the desired outcome." (Anthropic, ACE framework)

---

## agentskills.io Specification (Required Fields)

```yaml
---
name: skill-name-with-hyphens   # ≤64 chars, lowercase, hyphens only, gerund preferred
description: Use when [trigger conditions only]. [Specific symptoms/contexts].  # ≤1024 chars, third-person
---
```

**Only two required fields.** Optional: `license`, `compatibility`, `metadata`, `allowed-tools`.

---

## The CSO (Claude Search Optimization) Critical Trap

**FATAL MISTAKE:** Summarizing the skill's workflow in the description.

When a description summarizes steps/workflow, Claude follows the description as a shortcut and **never reads the full skill body**. This was observed and documented:
- Description saying "code review between tasks" → Claude did ONE review
- Same skill with description as pure trigger → Claude read flowchart and did TWO reviews correctly

**Rule:** Description = triggering conditions ONLY. Never include process/workflow summary.

```yaml
# ❌ BAD — summarizes workflow, Claude skips skill body
description: Use for TDD - write test first, watch it fail, write minimal code, refactor

# ❌ BAD — summarizes process steps  
description: Use when executing plans - dispatches subagent per task with code review between tasks

# ✅ GOOD — triggers only, no workflow
description: Use when implementing any feature or bugfix, before writing implementation code

# ✅ GOOD — triggers with symptoms
description: Use when tests have race conditions, timing dependencies, or pass/fail inconsistently
```

---

## Optimal SKILL.md Structure

```markdown
---
name: [gerund-form-name]
description: Use when [specific triggers/symptoms, no workflow summary]
---

# Skill Name

## Overview (2-3 sentences)
Core principle. What problem this solves.

## When to Use
[Small flowchart ONLY if decision non-obvious; otherwise bullet list of symptoms]
When NOT to use.

## Core Pattern / Process
[3-5 phases/steps with clear sequence]
[Numbered lists > prose for processes]

## Quick Reference (table)
[Scannable reference — most accessed section]

## Common Mistakes / Red Flags
[Anti-patterns + one-line fix each]

## Rationalizations Table (discipline skills only)
| Excuse | Reality |
|--------|---------|

## Related Skills / Supporting Files
```

---

## Token Budget Targets

| Skill Type | Target |
|-----------|--------|
| Getting-started / frequently-loaded | <150 words |
| Technique/pattern skill | <500 words |
| Complex workflow skill | <1500 words |
| Heavy reference (move to supporting file) | 100+ lines → separate .md |

**When to split into supporting files:**
- API reference / tool inventory > 100 lines → `reference.md`
- Reusable executable scripts → `.sh`, `.ts`, `.py`
- Worked examples > 50 lines → `examples/`

---

## Token Efficiency Techniques

### 1. Tables over prose
```markdown
# ❌ BAD (verbose)
search_symbols supports a query parameter for searching by name or signature. 
You can also pass kind to filter by symbol type, language to filter by language...

# ✅ GOOD (table)
| Param | Values | Notes |
|-------|--------|-------|
| query | string | name, signature, summary |
| kind | function/class/method | filter |
```

### 2. Cross-references instead of repetition
```markdown
# ❌ BAD
When dispatching subagents, create a TodoWrite task per agent, then...
[20 lines explaining subagent dispatch]

# ✅ GOOD
Use superpowers:dispatching-parallel-agents for workflow.
```

### 3. Conditional disclosure
```markdown
Only for multi-component systems:
[10 lines of detail skipped in single-component case]
```

### 4. Assume Claude's baseline knowledge
- Don't explain what Python imports are
- Don't explain what a function signature is
- Don't explain what JSON is
- Do explain: tool-specific gotchas, non-obvious sequences, hidden constraints

---

## Naming Conventions

| Pattern | Example | Anti-pattern |
|---------|---------|--------------|
| Gerund form | `using-jcodemunch` | `jcodemunch-usage` |
| Verb-first | `create-skill` | `skill-creation` |
| Specific | `condition-based-waiting` | `async-helpers` |
| No vague nouns | `systematic-debugging` | `debugging-utils` |

---

## Skill Types and Testing Approach

| Type | Examples | Test With |
|------|---------|-----------|
| Discipline-enforcing | TDD, verification-before-completion | Pressure scenarios with combined pressures |
| Technique | condition-based-waiting, root-cause-tracing | Application scenarios with variations |
| Pattern/mental model | systematic-debugging, brainstorming | Recognition + application scenarios |
| Reference | API docs, tool guides | Retrieval + application of specific information |

**Iron Law (TDD for Skills):** No skill without baseline test first. Watch agent fail WITHOUT skill → write minimal skill → verify compliance → close loopholes.

---

## Description Writing Formula

**Template:** `Use when [specific situation or task type]. [Optional: specific symptoms or tool context].`

**Examples from high-quality superpowers skills:**
- `Use when encountering any bug, test failure, or unexpected behavior, before proposing fixes`
- `Use when implementing any feature or bugfix, before writing implementation code`
- `Use when you have a spec or requirements for a multi-step task, before touching any code`
- `Use when facing 2+ independent tasks that can be worked on without shared state`

**Length target:** 100-200 characters is ideal. Under 500 is acceptable.

---

## Flowchart Usage Rules

**Use flowcharts ONLY for:**
- Non-obvious decision points
- Process loops where agent might stop too early
- "When to use A vs B" choices

**Never use for:** reference material (tables), linear instructions (numbered lists), code examples (code blocks)

**Format:** Graphviz DOT syntax, rendered via `render-graphs.js`

---

## File Organization Patterns

```
# Self-contained (most skills)
skill-name/
  SKILL.md

# With reusable tool
skill-name/
  SKILL.md
  helper-script.ts

# Heavy reference
skill-name/
  SKILL.md          # Overview + when-to-use + quick ref
  reference.md      # 100+ line API/tool reference
  examples/         # Worked examples
```

**Flat namespace** — all skills in one searchable directory.

---

## Quality Checklist

- [ ] Description: "Use when [triggers]", third-person, NO workflow summary
- [ ] Name: gerund, hyphens only, ≤64 chars
- [ ] SKILL.md under 500 lines
- [ ] Heavy reference (>100 lines) in separate file
- [ ] Tables instead of multi-paragraph prose
- [ ] Cross-references to other skills instead of repetition
- [ ] One excellent code example (not multi-language)
- [ ] Common mistakes / red flags section
- [ ] Tested: baseline failure documented + compliance verified

---

## Sources
- agentskills.io/specification
- anthropic.com (platform.claude.com/docs skill authoring best practices)
- anthropic.com/engineering/effective-context-engineering-for-ai-agents
- superpowers skills: writing-skills/SKILL.md + anthropic-best-practices.md (located at: /Users/luke/.claude/plugins/cache/superpowers-dev/superpowers/5.0.6/skills/writing-skills/)
- Observed behavior: CSO description trap documented in writing-skills/SKILL.md

---

## Evolution Log
- 2026-04-17: Created from research across agentskills.io spec, Anthropic platform docs, superpowers skill analysis, and context engineering paper (ACE, UC Berkeley/Stanford).