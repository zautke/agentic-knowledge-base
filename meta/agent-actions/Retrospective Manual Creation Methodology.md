---
title: Retrospective Manual Creation Methodology
type: note
permalink: meta/agent-actions/retrospective-manual-creation-methodology
tags:
- methodology
- retrospective
- documentation
- pattern-extraction
- globally-applicable
---

# Retrospective Manual Creation Methodology

A Scrum Master-inspired approach for extracting reusable patterns and creating instructional documentation from successful execution sessions.

## Purpose

Transform successful task executions into reusable, agent-facing instructional manuals through systematic retrospective analysis.

## The Retrospective Framework

### Phase 1: Session Analysis

Analyze the complete execution session asking:

1. **What Went Well?**
   - Identify successful patterns
   - Note smooth transitions
   - Recognize effective decisions

2. **What We Learned?**
   - Document discoveries
   - Capture edge cases encountered
   - Record unexpected behaviors

3. **What to Replicate?**
   - Extract repeatable patterns
   - Identify universal principles
   - Distill core methodologies

### Phase 2: Pattern Extraction

For each successful action, extract:

- **The Pattern**: What was done
- **The Context**: When to apply it
- **The Rationale**: Why it works
- **The Variations**: How to adapt it

### Phase 3: Generalization

Transform project-specific knowledge into universal principles:

| Project-Specific | Generalized |
|------------------|-------------|
| "Clear textarea with uid=3_2" | "Clear input element using fresh UID from snapshot" |
| "Port 5200" | "Verify correct port from project configuration" |
| "React onChange" | "Framework-specific event dispatching" |

### Phase 4: Documentation Structure

Create documentation with:

1. **Purpose Statement**: What the manual enables
2. **Prerequisites**: What must be in place
3. **Core Pattern**: The repeatable process
4. **Variations**: Framework/context adaptations
5. **Pitfalls**: Common mistakes and solutions
6. **Verification**: How to confirm success

## Quality Criteria for Manuals

### Signal-to-Noise Ratio
- Every sentence must add value
- Remove redundant explanations
- Prioritize actionable instructions

### Idempotent Execution
- Following the manual produces consistent results
- No hidden dependencies
- Clear success criteria

### Framework Agnosticism
- Principles apply across technologies
- Specific examples are clearly marked as such
- Adaptation guidance provided

### Progressive Disclosure
- Start with core pattern
- Add complexity gradually
- Advanced topics at end

## The Extraction Process

### Step 1: Capture Raw Session
Document every action, decision, and outcome during execution.

### Step 2: Identify Success Factors
What made this execution successful?

### Step 3: Abstract Patterns
Remove project-specific details, keep universal principles.

### Step 4: Validate Completeness
Could another agent replicate this with only the manual?

### Step 5: Test Documentation
Execute using only the manual, note gaps.

### Step 6: Iterate
Refine based on test execution.

## Example Transformation

### Raw Session Note:
```
Took snapshot, got uid=3_2 for textarea.
Cleared with evaluate_script.
Filled with markdown content.
Timeout occurred but content was there.
Took screenshot.
```

### Extracted Pattern:
```
1. Fresh snapshot → identify target UID
2. Clear element with event dispatch
3. Fill with test content
4. Verify with snapshot (ignore timeouts)
5. Document with screenshot
```

### Generalized Instruction:
```
The Core Test Pattern:
1. Take fresh snapshot to get current UIDs
2. Clear target element using framework-appropriate event dispatch
3. Input test content via fill operation
4. Take verification snapshot (timeouts are informational, not failures)
5. Capture screenshot for documentation
```

## Relations

- implements [[Knowledge Extraction]]
- applies_to [[Documentation Creation]]
- uses [[Scrum Retrospective]]
- related_to [[Pattern Recognition]]
- enables [[Reusable Methodologies]]
