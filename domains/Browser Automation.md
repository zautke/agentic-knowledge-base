---
title: Browser Automation
type: note
permalink: domains/browser-automation
---

# Browser Automation Domain

## Overview
This domain covers the automated testing, navigation, and interaction with web applications using agentic tools, specifically focusing on Chrome DevTools Protocol (CDP) and Model Context Protocol (MCP) integrations.

## Key Tools
- **Chrome DevTools MCP**: Allows agents to inspect, debug, and automate Chrome via Puppeteer/CDP.
- **Puppeteer**: High-level API to control Chrome or Chromium over the DevTools Protocol.
- **Playwright**: Modern end-to-end testing library with cross-browser support.

## Core Principles
1. **Natural Language Automation**: Using AI-powered element location (find_and_click) rather than brittle CSS selectors.
2. **Runtime Verification**: Always verifying the live state of the application via screenshots and console logs.
3. **Sandbox Awareness**: Running browser instances outside of restricted environments when deep automation is required.

## Project Context
- **Dev Server**: Vite-based demo running at `http://localhost:5173`.
- **Primary Goal**: Automated UI/UX showcase and theme validation.
- **Audit Findings**: Identified major version gaps in Vite (v4) and React (v18) that may impact automation stability.

## Best Practices
- Prefer `find_and_click` for UI interactions to increase resilience to DOM changes.
- Record performance traces for critical user journeys.
- Document specific selector strategies for complex shadow DOM or canvas elements.