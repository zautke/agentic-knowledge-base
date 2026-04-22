---
title: Autonomous Browser Testing Manual
type: note
permalink: meta/agent-actions/autonomous-browser-testing-manual
tags:
- testing
- browser-automation
- agent-protocol
- globally-applicable
- methodology
---

# Autonomous Browser Testing Manual

A comprehensive, framework-agnostic guide for AI agents to execute browser-based user-impersonation testing from inception to completion.

## Purpose

This manual enables AI agents to autonomously test web applications by simulating real user interactions through browser automation tools, verifying application behavior through accessibility tree analysis.

## Core Philosophy

- **User Impersonation**: Test as a real user would interact
- **Accessibility-First Verification**: Use accessibility tree snapshots as source of truth
- **Systematic Execution**: Follow consistent patterns for reproducible results
- **Graceful Error Handling**: Timeouts and errors are information, not failures

## Prerequisites

Before testing any web application:

1. **Server Running**: Verify dev server is active and responding
2. **Browser Connected**: Confirm browser automation tools are available
3. **Initial Navigation**: Navigate to application URL
4. **Baseline Snapshot**: Capture initial accessibility tree state

## The Core Test Pattern

Every test follows this 8-step pattern:

### Step 1: Fresh Snapshot
```
take_snapshot() → Get current accessibility tree with UIDs
```

### Step 2: Identify Target Element
```
Find element UID from snapshot (e.g., textarea, button, input)
```

### Step 3: Clear/Prepare Element
```javascript
// For React applications - dispatch events with bubbles: true
evaluate_script({
  function: "(el) => { el.value = ''; el.dispatchEvent(new Event('input', { bubbles: true })); return 'Cleared'; }",
  args: [{"uid": "TARGET_UID"}]
})
```

### Step 4: Input Test Content
```
fill({ uid: "TARGET_UID", value: "test content" })
```

### Step 5: Verification Snapshot
```
take_snapshot() → Verify content rendered correctly
```

### Step 6: Visual Documentation
```
take_screenshot({ filePath: "test-results/XX-test-name.png" })
```

### Step 7: Console Check
```
list_console_messages() → Check for errors
```

### Step 8: Update Progress
```
Mark test complete, move to next test
```

## Critical Success Patterns

### Pattern 1: Snapshot Lifecycle Management
```
Fresh Snapshot → Interact → STALE Snapshot → Fresh Snapshot → Interact
```
**Rule**: ALWAYS take fresh snapshot after ANY DOM interaction. UIDs become stale immediately.

### Pattern 2: Framework-Specific Event Dispatching

**React**:
```javascript
el.dispatchEvent(new Event('input', { bubbles: true }));
```

**Vue**:
```javascript
el.dispatchEvent(new Event('input', { bubbles: true }));
el.dispatchEvent(new Event('change', { bubbles: true }));
```

**Angular**:
```javascript
el.dispatchEvent(new Event('input', { bubbles: true }));
el.dispatchEvent(new Event('blur', { bubbles: true }));
```

### Pattern 3: Timeout ≠ Failure
```
fill() → timeout → take_snapshot() → verify content
```
**Rule**: Timeouts mean "waiting for response timed out", not "operation failed". Always verify actual state.

### Pattern 4: Accessibility Tree as Source of Truth
- **Visual Screenshot**: For human review only
- **Accessibility Tree**: For programmatic verification
- Check for expected elements, text content, states (checked, disabled, etc.)

### Pattern 5: Progressive Task Management
- One test in_progress at a time
- Immediate completion marking after verification
- Clear status visibility throughout

## Common Pitfalls and Solutions

### Pitfall 1: Stale Snapshot UIDs
**Symptom**: "This uid is coming from a stale snapshot"
**Solution**: Take fresh snapshot before every interaction

### Pitfall 2: Fill Timeouts
**Symptom**: "Timed out after waiting 5000ms"
**Solution**: Ignore timeout, verify with snapshot - content usually filled successfully

### Pitfall 3: React State Not Updating
**Symptom**: Input value changes but component doesn't re-render
**Solution**: Dispatch events with `{ bubbles: true }`

### Pitfall 4: Wrong Port
**Symptom**: Connection refused or wrong application
**Solution**: Verify correct port in project configuration

### Pitfall 5: Element Not Found
**Symptom**: UID not in snapshot
**Solution**: Check if element is conditionally rendered, wait for it to appear

## Test Categories

### Functional Tests
- Basic input/output
- Form interactions
- Navigation
- State management
- Error handling

### Content Rendering Tests
- Text formatting
- Code blocks
- Lists and tables
- Media elements
- Special characters

### Edge Case Tests
- Empty content
- Invalid input
- Special characters
- Unicode/emoji
- XSS attempts

### Integration Tests
- Multiple component interaction
- API calls
- State persistence
- Real-time updates

## Verification Checklist

For each test, verify:
- [ ] Expected elements present in accessibility tree
- [ ] Correct text content rendered
- [ ] Proper element states (checked, disabled, expanded)
- [ ] No unexpected console errors
- [ ] Visual appearance matches expectations (screenshot)

## Error Recovery

### If Test Fails
1. Document the failure with screenshot
2. Check console for errors
3. Verify element exists in DOM
4. Check for timing issues
5. Retry with fresh snapshot

### If Browser Disconnects
1. Re-establish browser connection
2. Navigate to application
3. Take fresh snapshot
4. Resume from last successful test

## Relations

- applies_to [[Browser Automation Testing]]
- uses [[Chrome DevTools MCP]]
- implements [[User Impersonation Testing]]
- related_to [[Accessibility Testing]]
