---
title: Browser Extension CDP Testing Patterns
type: reference
permalink: reference/browser-automation/browser-extension-cdp-testing-patterns
tags:
- domain/browser-automation
- domain/cdp
- project/agent-kb
---

# Browser Extension CDP Testing Patterns

ROLE: You are a document curator for Browser Extension CDP Testing Patterns -- an evolving reference that documents patterns and anti-patterns for testing Chrome extensions via Chrome DevTools Protocol (CDP). This document is a living artifact that improves through structured use.

GOVERNING PRINCIPLES:
1. ACCUMULATE, NEVER SUMMARIZE AWAY
2. STRUCTURED INCREMENTAL UPDATES ONLY
3. EVIDENCE-LINKED ENTRIES
4. SELF-REFERENTIAL VERIFICATION
5. EVOLUTION LOG IS MANDATORY
6. TRIGGER CONDITIONS FOR UPDATE:
   - a. A new CDP testing failure occurs due to environment or context issues.
   - b. A new method for injecting or evaluating code within extension service workers is discovered.
7. WHAT NOT TO CHANGE: Do not modify this metaprompt. Do not remove entries from registries (mark as deprecated instead). Do not change section numbering.
8. QUALITY SIGNAL TRACKING: Record whether these patterns successfully guided a debugging session or if the agent had to improvise.

---

## 1. Anti-Pattern Registry

| Date | Anti-Pattern | Context & Example | Impact | Source |
|------|-------------|-------------------|--------|--------|
| 2026-03-08 | Testing internal background logic exclusively via UI | UI states often depend on external factors (e.g., Docker containers or API keys). When an external service is down, the React UI may disable features (like the Send button), preventing the agent from testing the isolated background logic (`chrome.runtime.sendMessage`). | False negatives; the agent concludes the extension is broken when only the UI is locked out. Time wasted debugging UI state instead of testing the invariant. | WXT LLM Bridge Testing Session |
| 2026-03-08 | Using `Target.activateTarget` from a page-level CDP session | Attempting to activate a tab while connected to a specific page's WebSocket debugger URL yields a `Not allowed` error. | The script crashes; tab activation fails. | WXT LLM Bridge Testing Session |

## 2. Technique Library

### 2.1. Isolated Background Testing via CDP
When a UI blocks interaction due to missing external dependencies (e.g. LLM provider offline), bypass the UI entirely to test the extension's background invariants.
- **Method**: Use a native Node.js CDP script (`node --input-type=module`) to connect to the extension's sidepanel or popup context.
- **Execution**: Run `Runtime.evaluate` in the sidepanel context to directly invoke `chrome.runtime.sendMessage(...)` and simulate what the UI *would* have done.

### 2.2. Service Worker Message Spying
To verify message-passing invariants (e.g. ensuring a message is only sent once per cache hit), you can inject a counter into the Service Worker.
- **Method**: Connect CDP to the Service Worker target.
- **Execution**: Use `Runtime.evaluate` to wrap or augment the global `chrome.runtime.onMessage.addListener`.
- **Example**:
  ```javascript
  globalThis.__msgCounts = { HASH: 0, CTX: 0 };
  chrome.runtime.onMessage.addListener(function(msg) {
    if (msg.type === 'GET_HASH') globalThis.__msgCounts.HASH++;
    return false; // Let existing handlers process it
  });
  ```

### 2.3. Browser-Level CDP Connections for Target Commands
To manipulate tabs (e.g., `Target.activateTarget`, `Target.createTarget`), you must connect to the **browser-level** WebSocket debugger URL, not a page-level one.
- **Method**: Fetch `http://127.0.0.1:9222/json/version` and connect to the `webSocketDebuggerUrl` provided there (e.g., `ws://.../devtools/browser/uuid`).

---

## 3. Verification Boundaries for Extension Tool Loops

When implementing agentic tool loops in a Chrome MV3 extension, unit tests and real-browser tests cover different layers. The boundary is the `browser.runtime.sendMessage` call.

### What unit tests can fully verify

| Component | How | Coverage |
|-----------|-----|----------|
| Tool loop control flow (rounds, stop conditions) | Inject mock `chat` fn | Full |
| Error handling in `toolExecutor` (throws, `is_error`) | Inject mock `toolExecutor` | Full |
| Loop exhaustion / throw after max rounds | Mock `chat` always returning tool_use | Full |
| Provider wire format conversion (`toAnthropicTools`, `toOpenAITools`) | Pure function tests | Full |
| Non-streaming tool loop branch triggering | Check `tools?.length && toolExecutor` guard | Full |

### What requires real Chrome (not unit-testable)

| Component | Why browser required |
|-----------|---------------------|
| `EXECUTE_TOOL` message routing | `browser.runtime.sendMessage` is Chrome runtime |
| `chrome.tabs.query({ active: true })` | Requires live tab context |
| `chrome.debugger.attach/sendCommand` | CDP requires a real debuggee |
| Content script messaging | Real tab + injected content script |
| Dexie persistence across extension reloads | Real IndexedDB across service worker restarts |

### Anti-pattern: claiming tool loop is "tested" after unit tests only

Declaring the active tab context milestone complete after Vitest passes is accurate for the logic layer but leaves the full chain (`useAgentLoop → sendMessage → ToolRegistry → WebContextProvider → Chrome API`) untested. The integration gap is:

1. Build: `pnpm build` — no type errors in production bundle
2. Load unpacked: `chrome://extensions → Load unpacked → dist/chrome-mv3`
3. Enable DevTools toggle, navigate to any page, ask "what page am I on?" — verify system prompt injection
4. Ask "click the first button on this page" — verify `click_element` tool call completes via Ollama
5. Reload extension, re-open sidepanel — verify DevTools toggle is still ON (Dexie persistence)

**Source**: WXT LLM Bridge active tab context milestone, 2026-04-21

### Playwright headless results (2026-04-21)

Playwright 1.58.2 + headless Chromium 145 + `launchPersistentContext --load-extension` was used for Scenarios 1–3. Results:
- Extension loads, SW registers: **PASS**
- Sidepanel UI renders (hamburger, wrench, gear): **PASS**
- Dexie DevTools toggle persistence across page reload: **PASS**
- Message send + system prompt injection: **PARTIAL** (no Ollama in sandbox)
- `get_tab_info` tool call: **SKIP** (Ollama not reachable at localhost:11434)

Playwright's `launchPersistentContext` with `--load-extension` and `--disable-extensions-except` flags is the reliable headless Chrome extension loading pattern for MV3.

### Full live run results (2026-04-21, remote Ollama endpoint)

**Model:** `deepseek-v3.1:671b-clo` via `https://ollama.braisenly.com`

| # | Scenario | Result | Elapsed |
|---|---|---|---|
| 1 | Extension loads, no fatal errors | PASS | 6.8s |
| 2 | Sidepanel renders core controls | PASS | 4.5s |
| 3 | Configure Ollama base URL | PASS | 7.8s |
| 4 | DevTools toggle + Dexie persist | PASS | 6.0s |
| 5 | Provider connection poll | PASS | **5s elapsed** |
| 6 | Tab context injection — "what page am I on?" | PASS | Response: "Example Domain page at https://example.com/" |
| 7 | Tool call — "what is the title of the current tab?" | PASS | Response: "The title of the current tab is 'Example Domain'" |

**Provider connection timing**: 5 seconds with remote endpoint `https://ollama.braisenly.com`. Local Ollama (`localhost:11434`) may take 60+ seconds — always poll for 180s minimum when testing local.

**Playwright locator bug found**: `sidepanel.locator('li.justify-start')` does not match assistant message bubbles. Correct locator: `sidepanel.locator('li').filter({ has: sidepanel.locator('.items-start') }).last()` or target the message text div directly.

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-03-08 | Initial Creation | Documented critical traps encountered when testing WXT extensions via Playwright/CDP, specifically the UI-lockout trap and the Target domain restriction. | Failed test run due to offline Docker container hiding valid background logic. |
