---
title: Testing Manifesto
created: 2026-02-16
modified: 2026-02-16
type: instruction
entity_type: instruction
status: evergreen
permalink: instructions/testing-manifesto
tags:
- project/agent-kb
- domain/testing
- domain/vitest
- domain/react
- domain/nextjs
- status/evergreen
- op/create-node
- source/largo
- machine/largo
---

# Testing Manifesto

A project-agnostic, comprehensive testing strategy for React/Next.js applications using Vitest. This is a living document maintained under the [[Agentic Living Document Playbook]] and governed by the [[Document Curation Metaprompt]].

**Applicable to:** Any React 19 + Next.js 16 + Redux Toolkit + MUI application using Vitest 4.x.
**First applied to:** Claude Export Viewer (2026-02-16).

---

## Agent Curation Metaprompt

```
ROLE: You are a document curator for the Testing Manifesto -- an evolving
instructional document that defines a comprehensive testing strategy for
React/Next.js applications. This document is a living artifact that improves
through structured use across multiple projects.

GOVERNING PRINCIPLES:
1. ACCUMULATE, NEVER SUMMARIZE AWAY
2. STRUCTURED INCREMENTAL UPDATES ONLY
3. EVIDENCE-LINKED ENTRIES (date, context, example, source)
4. SELF-REFERENTIAL VERIFICATION (check before adding)
5. EVOLUTION LOG IS MANDATORY (append-only)
6. TRIGGER CONDITIONS FOR UPDATE:
   a. The manifesto is applied to a new project and reveals a gap
   b. A new Vitest major version ships with breaking changes or new features
   c. A test anti-pattern is discovered in production
   d. Research surfaces a better approach to an existing technique
   e. A testing tool in the stack is deprecated or replaced
   f. The user explicitly requests an update
   g. Test run persistence reveals a pattern (flaky tests, slow suites, coverage gaps)
7. WHAT NOT TO CHANGE:
   - Do not modify this metaprompt without user consent
   - Do not remove entries from Anti-Pattern Registry (deprecate instead)
   - Do not change section numbering
   - Do not remove techniques from the Technique Library
8. QUALITY SIGNAL TRACKING (after each application, record which sections
   were used, which were skipped, and which had insufficient guidance)
```

---

## 1. Purpose & Scope

### 1.1 Purpose

This manifesto defines a **dual-track, multi-agent testing strategy** for React/Next.js applications. It provides:
- A testing philosophy grounded in the Testing Trophy and SMURF frameworks
- Concrete Vitest 4.x configuration blueprints
- A multi-agent roster for dividing test implementation work
- An iterative, task-based implementation plan
- Test persistence and efficacy tracking patterns

### 1.2 Scope

**In scope:**
- Unit testing (pure functions, Redux slices, hooks)
- Component testing (React components with @testing-library/react)
- Browser integration testing (Vitest Browser Mode + Playwright)
- Accessibility testing (vitest-axe)
- E2E testing (Playwright)
- Visual regression testing (Vitest Browser Mode toMatchScreenshot)
- Test persistence, reporting, and trend analysis

**Out of scope:**
- Load/stress testing
- Security penetration testing
- Manual QA procedures

---

## 2. Testing Philosophy

### 2.1 The Testing Trophy (Kent C. Dodds, 2018)

The testing trophy inverts the traditional testing pyramid for frontend applications:

```
          ___
         / E2E \          <- Small: Critical user journeys only
        /________\
       /          \
      / Integration \     <- LARGEST: Components as users use them
     /______________\
    /                \
   /   Unit Tests     \   <- Medium: Pure functions, reducers, utils
  /____________________\
 /                      \
/ Static Analysis (TS)   \  <- Base: TypeScript strict, ESLint
/________________________\
```

**Core principle:** Integration tests provide the highest confidence-per-dollar. Test components as users interact with them, not as code is structured.

### 2.2 SMURF Framework (Google, Adam Bender, Oct 2024)

Rather than prescribing a test shape, evaluate each test type across 5 axes:

| Axis | Description |
|------|-------------|
| **S**peed | How fast does the test execute? |
| **M**aintainability | How much effort to keep the test passing as code changes? |
| **U**tilization | How often does the test catch real bugs? |
| **R**eliability | How often does the test produce false positives/negatives? |
| **F**idelity | How closely does the test environment match production? |

**Application:** For each test type in your suite, draw a radar chart across these 5 axes. Optimize the portfolio, not individual tests.

### 2.3 Guiding Principles

1. **Test behavior, not implementation.** Query by ARIA roles and visible text, not CSS classes or component internals.
2. **Prefer integration over isolation.** Render with real Redux stores and theme providers. Do not mock `useSelector` or `useDispatch`.
3. **Use the real thing when practical.** Vitest Browser Mode for tests that need real CSS, layout, or browser APIs.
4. **Every test must justify its existence.** A test that never catches a bug is maintenance overhead.
5. **Flaky tests are worse than no tests.** A flaky test trains developers to ignore failures.

---

## 3. Dual-Track Testing Architecture

### 3.1 Track Overview

```
PR opened/pushed → ┬→ [Track 1: Fast Feedback]     → MUST pass to merge
                    └→ [Track 2: High-Fidelity]     → Report-only (initially)
                                                       Graduate to blocking
```

### 3.2 Track 1: Fast Feedback

| Property | Value |
|----------|-------|
| **Scope** | Unit tests + Component tests + Accessibility scans |
| **Environment** | jsdom (simulated DOM) |
| **Target time** | < 30 seconds |
| **Merge gate** | Yes -- blocks merge if failing |
| **Runner** | `vitest --project unit --project component --project a11y` |

### 3.3 Track 2: High-Fidelity

| Property | Value |
|----------|-------|
| **Scope** | Browser integration tests + Visual regression + E2E |
| **Environment** | Real Chromium (Vitest Browser Mode) + Playwright |
| **Target time** | < 5 minutes |
| **Merge gate** | No (initially) -- report asynchronously. Graduate to blocking when stable. |
| **Runner** | `vitest --project browser` + `playwright test` |

### 3.4 Selective Execution

- Use Vitest's `--changed` flag to run only tests affected by changed files on pre-push hooks.
- Full suite runs on PR creation and merge to main.
- Sharded runs for large suites: `vitest run --shard=1/3 --reporter=blob`, merge with `vitest --merge-reports`.

---

## 4. Technology Stack

### 4.1 Core Testing Tools

| Tool | Version | Purpose |
|------|---------|---------|
| Vitest | 4.x | Test runner (unit, component, browser) |
| @testing-library/react | Latest | Component rendering and queries |
| @testing-library/dom | Latest | DOM queries |
| @testing-library/jest-dom | Latest | Custom DOM matchers |
| @testing-library/user-event | Latest | Simulated user interactions (jsdom only) |
| vitest-browser-react | Latest | Browser Mode component rendering |
| @vitest/browser-playwright | Latest | Browser Mode provider |
| vitest-axe | Latest | Accessibility testing (axe-core) |
| msw | Latest | API mocking (Mock Service Worker) |
| @vitest/coverage-v8 | Latest | Code coverage (V8 provider) |

### 4.2 Reporting Tools

| Tool | Purpose |
|------|---------|
| Vitest JSON reporter | Machine-readable test results |
| Vitest JUnit reporter | CI/CD integration (Jenkins, GitLab) |
| Vitest HTML reporter | Visual test report |
| CTRF reporter (`vitest-ctrf-json-reporter`) | Standardized cross-tool format |
| Allure Report (`allure-vitest`) | Visual dashboards with history |
| github-actions reporter | PR annotations in GitHub |

### 4.3 E2E Tools

| Tool | Purpose |
|------|---------|
| Playwright (`@playwright/test`) | Full E2E testing |
| Playwright Trace | Failure debugging (screenshots + DOM snapshots + network) |

---

## 5. Vitest Configuration Blueprint

### 5.1 Multi-Project Configuration

```typescript
// vitest.config.mts
import { defineConfig } from 'vitest/config'
import react from '@vitejs/plugin-react'
import tsconfigPaths from 'vite-tsconfig-paths'
import { playwright } from '@vitest/browser-playwright'

const isCI = process.env.CI === 'true'

export default defineConfig({
  plugins: [tsconfigPaths(), react()],
  test: {
    projects: [
      // Track 1: Unit tests (pure functions, reducers, hooks)
      {
        extends: true,
        test: {
          name: 'unit',
          include: ['src/**/*.test.ts', 'src/**/*.test.tsx'],
          exclude: ['src/**/*.browser.test.{ts,tsx}', 'src/**/*.a11y.test.{ts,tsx}'],
          environment: 'jsdom',
          globals: true,
          setupFiles: ['./vitest.setup.ts'],
        },
      },
      // Track 1: Accessibility tests (vitest-axe)
      {
        extends: true,
        test: {
          name: 'a11y',
          include: ['src/**/*.a11y.test.{ts,tsx}'],
          environment: 'jsdom', // NOT happy-dom (vitest-axe incompatibility)
          globals: true,
          setupFiles: ['./vitest.setup.ts', './vitest.a11y.setup.ts'],
        },
      },
      // Track 2: Browser integration tests
      {
        extends: true,
        test: {
          name: 'browser',
          include: ['src/**/*.browser.test.{ts,tsx}'],
          browser: {
            enabled: true,
            provider: playwright(),
            instances: [{ browser: 'chromium' }],
            trace: {
              mode: 'retain-on-failure',
              screenshots: true,
              snapshots: true,
            },
          },
        },
      },
    ],
    reporters: isCI
      ? ['dot', 'github-actions', ['json', { outputFile: './test-results/results.json' }], ['junit', { outputFile: './test-results/junit.xml' }]]
      : ['default'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html', 'lcov'],
      reportsDirectory: './test-results/coverage',
      include: ['src/**/*.{ts,tsx}'],
      exclude: ['src/**/*.test.{ts,tsx}', 'src/**/*.d.ts', 'src/app/layout.tsx'],
    },
  },
})
```

### 5.2 Setup Files

```typescript
// vitest.setup.ts
import '@testing-library/jest-dom/vitest'
```

```typescript
// vitest.a11y.setup.ts
import 'vitest-axe/extend-expect'
```

### 5.3 File Naming Conventions

| Convention | Test Type | Track |
|-----------|-----------|-------|
| `*.test.ts` | Unit test (pure function) | 1 |
| `*.test.tsx` | Component test (jsdom) | 1 |
| `*.a11y.test.tsx` | Accessibility test | 1 |
| `*.browser.test.tsx` | Browser integration test | 2 |
| `*.e2e.ts` (in `/e2e/`) | End-to-end test | 2 |

---

## 6. Test Utilities & Shared Infrastructure

### 6.1 renderWithProviders

The canonical test utility for rendering Redux-connected components:

```typescript
// src/test-utils/renderWithProviders.tsx
import React, { type PropsWithChildren } from 'react'
import { render, type RenderOptions } from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import { Provider } from 'react-redux'
import { configureStore, combineReducers } from '@reduxjs/toolkit'
import { ThemeProvider } from '@mui/material/styles'
import { CssBaseline } from '@mui/material'
import chatReducer from '@/store/slices/chatSlice'
import exportReducer from '@/store/slices/exportSlice'
import uiReducer from '@/store/slices/uiSlice'
import searchReducer from '@/store/slices/searchSlice'
import theme from '@/theme'
import type { RootState } from '@/store'

const rootReducer = combineReducers({
  chat: chatReducer,
  export: exportReducer,
  ui: uiReducer,
  search: searchReducer,
})

type SetupStoreOptions = {
  preloadedState?: Partial<RootState>
}

export function setupStore(options: SetupStoreOptions = {}) {
  return configureStore({
    reducer: rootReducer,
    preloadedState: options.preloadedState as RootState | undefined,
  })
}

type ExtendedRenderOptions = Omit<RenderOptions, 'wrapper'> & SetupStoreOptions & {
  store?: ReturnType<typeof setupStore>
}

export function renderWithProviders(
  ui: React.ReactElement,
  options: ExtendedRenderOptions = {}
) {
  const {
    preloadedState,
    store = setupStore({ preloadedState }),
    ...renderOptions
  } = options

  const Wrapper = ({ children }: PropsWithChildren) => (
    <Provider store={store}>
      <ThemeProvider theme={theme}>
        <CssBaseline />
        {children}
      </ThemeProvider>
    </Provider>
  )

  return {
    store,
    user: userEvent.setup(),
    ...render(ui, { wrapper: Wrapper, ...renderOptions }),
  }
}
```

### 6.2 Test Fixture Management

```typescript
// src/test-utils/fixtures.ts
import type { ExportedChat, ExportedMessage } from '@/store/slices/exportSlice'

export function createMockMessage(overrides: Partial<ExportedMessage> = {}): ExportedMessage {
  return {
    role: 'Response',
    time: '2026-01-15T10:30:00Z',
    say: 'Hello, this is a test message.',
    blocks: [{ type: 'text', content: 'Hello, this is a test message.' }],
    ...overrides,
  }
}

export function createMockChat(overrides: Partial<ExportedChat> = {}): ExportedChat {
  return {
    metadata: {
      title: 'Test Conversation',
      dates: {
        created: '2026-01-15T10:00:00Z',
        updated: '2026-01-15T11:00:00Z',
        exported: '2026-01-15T12:00:00Z',
      },
      link: '',
      powered_by: 'Claude',
    },
    messages: [
      createMockMessage({ role: 'Prompt', say: 'Hello' }),
      createMockMessage({ role: 'Response', say: 'Hi there!' }),
    ],
    ...overrides,
  }
}
```

---

## 7. UI Touchpoint Coverage Strategy

### 7.1 Touchpoint Manifest

Maintain a structured JSON manifest that catalogs every interactive element in the application:

```json
{
  "pages": [
    {
      "name": "Export View",
      "components": [
        {
          "name": "FileUploadArea",
          "interactions": [
            { "type": "drop", "element": "paper", "testId": "file-upload-area" },
            { "type": "dragOver", "element": "paper", "testId": "file-upload-area" },
            { "type": "dragLeave", "element": "paper", "testId": "file-upload-area" },
            { "type": "change", "element": "input[type=file]", "testId": "file-input" }
          ],
          "states": ["idle", "dragOver", "loading", "error"]
        }
      ]
    }
  ]
}
```

### 7.2 Coverage Tracking

1. **data-testid convention:** Tag all interactive elements with `data-testid` attributes following the pattern `{component}-{element}` (e.g., `file-upload-area`, `sidebar-toggle`).
2. **Gap detection:** After each test run, compare the touchpoint manifest against test file imports and `getByTestId` calls to identify untested interactions.
3. **Coverage matrix:** Maintain a `Component x Interaction -> Test File` mapping in the test results directory.

### 7.3 Custom Audit Script Pattern

```typescript
// scripts/audit-test-coverage.ts
// Crawl src/components/ for data-testid attributes
// Cross-reference against *.test.tsx files for getByTestId calls
// Output: untested touchpoints report
```

---

## 8. Multi-Agent Testing Roster

### 8.1 Agent Definitions

| Agent ID | Type | Role | File Convention | Environment | Track |
|----------|------|------|----------------|-------------|-------|
| **UNIT-1** | general | Redux slices: reducers, selectors, initial state, edge cases | `*.test.ts` in `store/slices/` | jsdom | 1 |
| **UNIT-2** | general | Utility functions: exportUtils, search helpers, formatters | `*.test.ts` in `utils/`, `lib/` | jsdom | 1 |
| **UNIT-3** | general | Custom hooks: useDebounce, useAppDispatch, useAppSelector | `*.test.ts` in `hooks/` | jsdom | 1 |
| **COMP-1** | general | Layout components: AppShell, Sidebar, ChatArea | `*.test.tsx` in `components/layout/` | jsdom | 1 |
| **COMP-2** | general | Chat components: ChatInput, MessageBubble, MessageList, EmptyState | `*.test.tsx` in `components/chat/` | jsdom | 1 |
| **COMP-3** | general | Export components: ExportViewer, FileUploadArea, ExportNavigationSidebar, ExportChatHeader, ExportMessageList, ExportMessageBubble | `*.test.tsx` in `components/export/` | jsdom | 1 |
| **COMP-4** | general | Atom components: CodeBlock, ThinkingBlock, ArtifactViewer, ArtifactPlaceholder, MessageContent, MarkdownRenderer | `*.test.tsx` in `components/chat/` | jsdom | 1 |
| **BROWSER-1** | general | Browser integration: MUI portals, drag-drop, keyboard shortcuts, CSS layout, responsive | `*.browser.test.tsx` | Chromium | 2 |
| **A11Y-1** | general | Accessibility: vitest-axe scans on every component, WCAG 2.1 AA compliance | `*.a11y.test.tsx` | jsdom | 1 |
| **E2E-1** | general | End-to-end: critical user journeys (upload, navigate, search, persist) | `*.e2e.ts` in `/e2e/` | Playwright | 2 |
| **HEAL-1** | general | Maintenance: flaky test triage, broken locator repair, test health monitoring | N/A | N/A | -- |

### 8.2 Agent Instructions Template

Each agent receives a prompt structured as:

```
You are testing agent {AGENT_ID}. Your role is {ROLE}.

SCOPE: You are responsible for testing {SCOPE_DESCRIPTION}.
FILES: Create test files matching the convention {FILE_CONVENTION}.
ENVIRONMENT: Your tests run in {ENVIRONMENT}.

CONSTRAINTS:
1. Use renderWithProviders for any component that uses Redux or MUI theme.
2. Query elements by ARIA roles and visible text, not CSS classes.
3. Do not mock useSelector or useDispatch. Use preloadedState instead.
4. Each test file must import from the shared test utilities.
5. Run `vitest --project {PROJECT} --reporter=json` after writing tests.
6. Report: total tests, passing, failing, coverage delta.

TOUCHPOINTS TO COVER:
{LIST_FROM_TOUCHPOINT_MANIFEST}
```

### 8.3 Agent Coordination Protocol

```
                    ┌────────────────────────────────────┐
                    │          ORCHESTRATOR               │
                    │  - Reads TASKS.md P3 section        │
                    │  - Assigns work to agents           │
                    │  - Runs vitest --reporter=json      │
                    │  - Collects/merges results          │
                    │  - Updates TASKS.md checkboxes      │
                    └──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──────┘
                       │  │  │  │  │  │  │  │  │  │
Phase 1 (parallel):  U1 U2 U3
Phase 2 (parallel):  C1 C2 C3 C4  A1
Phase 3:             BROWSER-1
Phase 4:             E2E-1
Ongoing:             HEAL-1

Each agent:
  1. Reads shared test utilities (renderWithProviders, fixtures)
  2. Reads touchpoint manifest for assigned components
  3. Writes test files following conventions
  4. Runs tests, reports results
  5. No cross-agent dependencies within a phase
```

### 8.4 Optimal Work Division

| Phase | Agents | Parallelism | Prerequisites |
|-------|--------|------------|---------------|
| **Foundation** | Orchestrator | Serial | Vitest config, setup files, test utilities, fixtures |
| **Phase 1: Unit** | UNIT-1, UNIT-2, UNIT-3 | Full parallel | Foundation complete |
| **Phase 2: Components** | COMP-1, COMP-2, COMP-3, COMP-4, A11Y-1 | Full parallel | Phase 1 complete (shared utils validated) |
| **Phase 3: Browser** | BROWSER-1 | Serial | Phase 2 complete (component tests establish baselines) |
| **Phase 4: E2E** | E2E-1 | Serial | Phase 3 complete |
| **Ongoing** | HEAL-1 | Async | Any test failures detected |

---

## 9. Iterative Implementation Plan

### 9.1 Foundation Tasks (Orchestrator)

```
[ ] Install dependencies:
    pnpm add -D vitest @vitejs/plugin-react @testing-library/react
    @testing-library/dom @testing-library/jest-dom @testing-library/user-event
    vitest-browser-react @vitest/browser-playwright vitest-axe
    @vitest/coverage-v8 vite-tsconfig-paths msw vitest-ctrf-json-reporter
[ ] Create vitest.config.mts (Section 5.1 blueprint)
[ ] Create vitest.setup.ts (Section 5.2)
[ ] Create vitest.a11y.setup.ts (Section 5.2)
[ ] Create src/test-utils/renderWithProviders.tsx (Section 6.1)
[ ] Create src/test-utils/fixtures.ts (Section 6.2)
[ ] Add scripts to package.json:
    "test": "vitest",
    "test:unit": "vitest --project unit",
    "test:a11y": "vitest --project a11y",
    "test:browser": "vitest --project browser",
    "test:coverage": "vitest --coverage",
    "test:ci": "vitest run --reporter=json --reporter=junit"
[ ] Verify foundation: run vitest with a single trivial test
```

### 9.2 Phase 1: Unit Tests

```
UNIT-1 (Redux slices):
[ ] chatSlice.test.ts -- 7 reducers, ~15 cases
[ ] exportSlice.test.ts -- 8 reducers + REHYDRATE, ~20 cases
[ ] uiSlice.test.ts -- 6 reducers, ~8 cases
[ ] searchSlice.test.ts -- expand existing 8 tests

UNIT-2 (Utilities):
[ ] exportUtils.test.ts -- 7 functions, ~30 cases
[ ] conversationParser.test.ts -- expand existing 5 tests with ~11 gap cases
[ ] search.test.ts -- expand existing 18 tests with ~7 gap cases

UNIT-3 (Hooks):
[ ] useDebounce.test.ts -- ~7 cases
```

### 9.3 Phase 2: Component + Accessibility Tests

```
COMP-1 (Layout):
[ ] AppShell.test.tsx -- Cmd+K handler, sidebar width, child rendering
[ ] Sidebar.test.tsx -- tabs, conversation list, hover actions, delete
[ ] ChatArea.test.tsx -- view routing, file upload pipeline, toolbar buttons

COMP-2 (Chat):
[ ] ChatInput.test.tsx -- Enter/Shift+Enter, disabled state, dispatch
[ ] MessageBubble.test.tsx -- user vs assistant, loading, timestamp
[ ] MessageList.test.tsx -- auto-scroll, empty state
[ ] EmptyState.test.tsx -- dispatch createConversation

COMP-3 (Export):
[ ] ExportViewer.test.tsx -- 3 state branches (empty, no selection, selected)
[ ] FileUploadArea.test.tsx -- drag-drop, validation, loading, error
[ ] ExportNavigationSidebar.test.tsx -- selection, delete, pluralization
[ ] ExportChatHeader.test.tsx -- dates, link, metadata
[ ] ExportMessageList.test.tsx -- empty state, message rendering
[ ] ExportMessageBubble.test.tsx -- Prompt vs Response, blocks vs fallback

COMP-4 (Atoms):
[ ] CodeBlock.test.tsx -- copy, language label, timeout reset
[ ] ThinkingBlock.test.tsx -- collapse/expand toggle
[ ] ArtifactViewer.test.tsx -- tabs, MIME mapping, fallback values
[ ] ArtifactPlaceholder.test.tsx -- prop rendering
[ ] MessageContent.test.tsx -- block type dispatch, null/empty handling
[ ] MarkdownRenderer.test.tsx -- all transforms, sanitization, edge cases

Common:
[ ] ErrorBoundary.test.tsx -- catch, recovery, fallback, dev/prod

A11Y-1 (Accessibility):
[ ] AppShell.a11y.test.tsx
[ ] Sidebar.a11y.test.tsx
[ ] FileUploadArea.a11y.test.tsx
[ ] SearchModal.a11y.test.tsx
[ ] ExportViewer.a11y.test.tsx
[ ] CodeBlock.a11y.test.tsx
[ ] (one per interactive component)
```

### 9.4 Phase 3: Browser Integration Tests

```
BROWSER-1:
[ ] SearchModal.browser.test.tsx -- MUI Dialog portal, focus management
[ ] FileUploadArea.browser.test.tsx -- real drag-drop events via CDP
[ ] AppShell.browser.test.tsx -- Cmd+K with real keyboard events
[ ] Sidebar.browser.test.tsx -- responsive behavior at breakpoints
[ ] ArtifactViewer.browser.test.tsx -- tab switching with real CSS transitions
[ ] Visual regression baselines for key components (toMatchScreenshot)
```

### 9.5 Phase 4: E2E Tests

```
E2E-1:
[ ] upload-json.e2e.ts -- Upload JSON -> sidebar update -> message render
[ ] upload-zip.e2e.ts -- Upload ZIP -> extract -> render
[ ] navigation.e2e.ts -- Select export -> view -> switch -> delete
[ ] search.e2e.ts -- Cmd+K -> query -> select result -> navigate
[ ] persistence.e2e.ts -- Upload -> refresh -> data persists
[ ] error-handling.e2e.ts -- Invalid file -> error message -> recovery
```

---

## 10. Test Persistence & Efficacy Tracking

### 10.1 Output Configuration

Every test run produces:

| Output | Location | Format | Purpose |
|--------|----------|--------|---------|
| Console summary | stdout | Text | Developer feedback |
| JSON results | `test-results/results.json` | Jest-compatible JSON | Machine-readable analysis |
| JUnit XML | `test-results/junit.xml` | JUnit | CI/CD integration |
| HTML report | `test-results/html/` | Interactive HTML | Visual review |
| Coverage report | `test-results/coverage/` | lcov + JSON + HTML | Coverage tracking |
| CTRF report | `test-results/ctrf.json` | CTRF JSON | Cross-tool standardized format |
| Playwright traces | `test-results/traces/` | ZIP (Playwright format) | Failure debugging |

### 10.2 Persistence Strategy

```
test-results/
  results.json        # Latest run (git-ignored)
  junit.xml           # Latest run (git-ignored)
  coverage/           # Latest coverage (git-ignored)
  history/            # Accumulated run summaries (committed)
    YYYY-MM-DD-HHmmss.json   # Snapshot per run
    summary.json              # Aggregated trends
```

### 10.3 Run Summary Schema

Each entry in `history/` contains:

```json
{
  "timestamp": "2026-02-16T14:30:00Z",
  "commit_sha": "abc123",
  "duration_ms": 12500,
  "total_tests": 150,
  "passed": 148,
  "failed": 1,
  "skipped": 1,
  "pass_rate": 0.9867,
  "coverage": {
    "statements": 72.5,
    "branches": 65.3,
    "functions": 80.1,
    "lines": 73.2
  },
  "flaky_tests": [],
  "new_failures": ["ExportViewer.test.tsx > renders error alert"],
  "fixed_tests": []
}
```

### 10.4 Flaky Test Detection

A test is classified as flaky when:
- Same test produces PASS and FAIL on the same commit SHA across different runs
- OR test fails then passes on retry within the same run (Vitest `retry` option)

**Protocol:**
1. Detected flaky tests are logged in `test-results/history/flaky-registry.json`
2. Flaky tests are NOT immediately quarantined -- they are investigated first
3. If root cause is non-deterministic (timing, animation, network), add `retry: 2` to the specific test
4. If root cause is environmental, fix the test or the environment
5. If unfixable, quarantine by moving to a `*.flaky.test.tsx` pattern and exclude from merge gate

### 10.5 Efficacy Metrics

Track these metrics across runs to evaluate the testing strategy:

| Metric | Calculation | Target |
|--------|-------------|--------|
| **Pass rate** | passed / total | >99% (after stabilization) |
| **Flaky test count** | tests in flaky-registry.json | 0 (ideal), <5 (acceptable) |
| **Coverage (statements)** | from v8 provider | >80% |
| **Coverage (branches)** | from v8 provider | >70% |
| **Mean test duration** | total_duration / total_tests | <100ms per test |
| **Suite duration (Track 1)** | unit + component + a11y | <30s |
| **Suite duration (Track 2)** | browser + e2e | <5min |
| **Defect escape rate** | bugs in production / total bugs | <10% |
| **Coverage delta per PR** | coverage_after - coverage_before | >= 0 (never regress) |

---

## 11. Anti-Pattern Registry

| # | Anti-Pattern | Context | Impact | Discovered | Source |
|---|-------------|---------|--------|-----------|--------|
| TAP-1 | **Using happy-dom with vitest-axe** | happy-dom's `isConnected` implementation is buggy, breaking axe-core's tree traversal | Accessibility tests produce false passes | 2026-02-16 | Vitest community, GitHub issues |
| TAP-2 | **Using @testing-library/user-event in Browser Mode** | Browser Mode has real CDP events; simulated events from userEvent conflict | Clicks don't fire, events are doubled | 2026-02-16 | Vitest docs, Epic Web Dev |
| TAP-3 | **Testing async Server Components with Vitest** | React 19 async RSC cannot be rendered in Vitest's test environment | Tests hang or throw | 2026-02-16 | Next.js 16 docs |
| TAP-4 | **Mocking useSelector/useDispatch in component tests** | Couples tests to Redux implementation; breaks on refactor | Tests pass but don't catch real bugs | 2026-02-16 | Redux docs (writing-tests) |
| TAP-5 | **Single environment for all test types** | jsdom can't test real CSS; Browser Mode is slow for unit tests | Either false confidence or slow suites | 2026-02-16 | Vitest docs (projects) |
| TAP-6 | **Page-level visual regression snapshots** | Entire page screenshots are brittle; unrelated changes cause failures | High false-positive rate | 2026-02-16 | Vitest visual regression docs |
| TAP-7 | **vi.spyOn on module exports in Browser Mode without vi.mock** | Browser Mode requires `vi.mock('./module', { spy: true })` workaround | Spies don't intercept | 2026-02-16 | Vitest Browser Mode docs |
| TAP-8 | **Ignoring flaky tests** | Developers learn to ignore all failures; real bugs slip through | Erosion of test suite trust | 2026-02-16 | Google Testing Blog, Atlassian Engineering |

---

## 12. Technique Library

| ID | Technique | Description | When to Use |
|----|-----------|-------------|-------------|
| TT-1 | **renderWithProviders** | Wraps component in Redux Provider + ThemeProvider. Returns store + user (userEvent). | Any component that uses `useSelector`, `useDispatch`, or MUI theme. |
| TT-2 | **Preloaded state injection** | Pass `preloadedState` to `setupStore` to set initial Redux state for a test. | Testing components that depend on specific Redux state. |
| TT-3 | **MSW handler factories** | Create reusable `http.get/post` handlers for Mock Service Worker. | Testing components that make API calls (future feature). |
| TT-4 | **Test fixture factories** | `createMockMessage()`, `createMockChat()` with spread overrides. | Any test that needs structured test data. |
| TT-5 | **Element-level visual regression** | `await expect(page.getByTestId('x')).toMatchScreenshot('name')` | Verifying visual appearance of specific components (not whole pages). |
| TT-6 | **Playwright Trace on failure** | `trace: { mode: 'retain-on-failure' }` in browser config. | Debugging failing browser tests. |
| TT-7 | **renderHook for custom hooks** | `renderHook(() => useDebounce(value, delay))` from RTL. | Testing hooks in isolation. |
| TT-8 | **vitest-axe component scanning** | `const results = await axe(container); expect(results).toHaveNoViolations()` | Every component's accessibility test. |
| TT-9 | **Fuse.js mock for search tests** | Mock Fuse constructor to control search results deterministically. | Testing search UI behavior without Fuse.js internals. |
| TT-10 | **File/Blob creation for upload tests** | `new File(['content'], 'test.json', { type: 'application/json' })` | Testing file upload handlers. |

---

## 13. Critical Gotchas & Known Issues

| # | Issue | Workaround | Affected Versions |
|---|-------|-----------|-------------------|
| CG-1 | Async Server Components cannot be unit tested | Use Playwright E2E for async RSC pages | React 19, Next.js 16 |
| CG-2 | MUI v7 + styled-components breaks in Vitest | Use Emotion (default MUI engine), or add package.json overrides | MUI v7, Vitest 4.x |
| CG-3 | Browser Mode `alert/confirm/prompt` auto-mocked | Use custom UI modals instead of native dialogs | Vitest 4.x Browser Mode |
| CG-4 | Browser Mode port defaults to 63315 | Configure via `browser.api.port` if conflicts | Vitest 4.x |
| CG-5 | V8 coverage with AST remapping (Vitest 3.2+) | No workaround needed; produces identical reports to Istanbul | Vitest 3.2+ |
| CG-6 | `@testing-library/react` `act()` warnings in React 19 | Usually safe to ignore; React 19 improved automatic batching | React 19, RTL 16+ |

---

## 14. Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-02-16 | Initial manifesto creation | First comprehensive testing strategy document. Derived from multi-agent research: Vitest 4.x docs, Redux testing docs, MUI testing guides, Kent C. Dodds' testing trophy, Google SMURF framework, Playwright Test Agents, Cypress UI Coverage, CTRF reporting standard. Applied to Claude Export Viewer codebase audit (~260 touchpoints across 31 files). | User request for comprehensive testing manifesto as project-agnostic living document. |
| 2026-02-16 | Metadata and link hygiene pass | Normalized frontmatter to the KB contract (`created`, `modified`, `entity_type`, `status`) and fixed relation-link formatting to remove unresolved graph targets. | KB hygiene sweep triggered by recent activity showing missing links. |
| 2026-02-16 | Relation keyword normalization | Standardized relation lines to canonical `verb [Target]` format to keep relation-type parsing consistent. | Follow-up graph hygiene pass. |

---

## 15. Research Sources

| # | Source | Type | Key Finding |
|---|--------|------|-------------|
| RS-1 | [Next.js 16 Testing with Vitest](https://nextjs.org/docs/app/guides/testing/vitest) | Official docs (2026-02-11) | Vitest is the officially recommended test runner for Next.js. Async RSC cannot be unit tested. |
| RS-2 | [Vitest 4.0 Announcement](https://voidzero.dev/posts/announcing-vitest-4) | Release notes (2025-10-22) | Browser Mode stable, visual regression, Playwright Trace support. |
| RS-3 | [Redux Writing Tests](https://redux.js.org/usage/writing-tests) | Official docs (2025-10-30) | Prefer integration tests. Don't mock hooks. renderWithProviders pattern. Now includes Vitest Browser Mode section. |
| RS-4 | [MUI Testing Guide](https://mui.com/material-ui/guides/testing/) | Official docs | Query by ARIA roles, not MUI internals. ThemeProvider wrapping required. |
| RS-5 | [Vitest Browser Mode](https://vitest.dev/guide/browser/) | Official docs (v4.0.17) | Real browser testing, CDP events, visual regression, trace support. |
| RS-6 | [Epic Web Dev: Vitest Browser Mode vs Playwright](https://www.epicweb.dev/vitest-browser-mode-vs-playwright) | Blog (2025-11) | Test code runs in browser (Vitest) vs Node.js (Playwright). Different use cases. |
| RS-7 | [Kent C. Dodds: Testing Trophy](https://kentcdodds.com/blog/write-tests) | Blog (2018, still canonical) | Integration tests give best confidence-per-dollar. |
| RS-8 | [Google SMURF](https://testing.googleblog.com/2024/10/smurf-beyond-test-pyramid.html) | Blog (2024-10) | 5-axis evaluation: Speed, Maintainability, Utilization, Reliability, Fidelity. |
| RS-9 | [Vitest Reporters](https://vitest.dev/guide/reporters) | Official docs | JSON, JUnit, HTML, blob, github-actions reporters. Sharded run merging. |
| RS-10 | [Playwright Test Agents](https://playwright.dev/docs/test-agents) | Official docs (2025-10) | Planner, Generator, Healer agents for E2E test lifecycle. |
| RS-11 | [Allure Report](https://allurereport.org/) | Tool | Visual test reporting with history tracking. |
| RS-12 | [CTRF Reporter](https://github.com/david2tm/d2t-vitest-ctrf-json-reporter) | Tool | Common Test Report Format for cross-tool analysis. |
| RS-13 | [Cypress UI Coverage](https://docs.cypress.io/ui-coverage/core-concepts/interactivity) | Competitor tool | Automatic interactive element discovery and coverage tracking. |
| RS-14 | [Martin Fowler: Diverse Shapes of Testing](https://martinfowler.com/articles/2021-test-shapes.html) | Essay (2021) | The argument about test shapes is opaque because terms mean different things to different people. |
| RS-15 | [Atlassian: Taming Test Flakiness](https://www.atlassian.com/blog/atlassian-engineering/taming-test-flakiness) | Engineering blog | Git-based + statistical flaky test detection at scale. |
| RS-16 | [Feature-Sliced Design: Frontend Testing Strategy](https://feature-sliced.design/uz/blog/frontend-testing-strategy) | Blog (2026) | Module boundaries should define test boundaries. |

---

## Relations

- specializes [[Agentic Living Document Playbook]]
- implements [[Document Curation Metaprompt]]
- related_to [[PRD Document Creation Playbook]]
- related_to [[Action Taxonomy – Index]]
