---
title: Agentic Browser Automation Overview
type: note
permalink: browser-automation/agentic-browser-automation-overview
---

# Agentic Browser Automation

## Overview
This domain focuses on autonomous agent interaction with web browsers for testing, data extraction, and workflow automation.

## Core Principles
1. **Evidence-Based Verification**: Every action must be verified through runtime state (DOM checks, screenshots, console logs).
2. **Context-Aware Navigation**: Agents should understand the UI hierarchy before interacting.
3. **Resilient Selectors**: Prefer accessible roles and labels over brittle CSS selectors.

## Tooling
- **Playwright**: Core automation framework.
- **Chrome DevTools Protocol (CDP)**: For deep inspection and performance auditing.
- **Vite**: Preferred dev server for rapid feedback loops.

## References
- [[Browser Extension CDP Testing Patterns]]

## Best Practices
- Always verify the server is healthy before running automation.
- Use `FUNCTIONAL_AUDIT_AND_APPROVAL.md` to track verification trails.
- Capture screenshots on failure.