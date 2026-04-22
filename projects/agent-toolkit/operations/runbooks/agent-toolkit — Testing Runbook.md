---
title: agent-toolkit — Testing Runbook
type: reference
permalink: agent-kb/projects/agent-toolkit/operations/runbooks/agent-toolkit-testing-runbook
created: 2026-04-18
modified: 2026-04-18
entity_type: reference
status: evergreen
tags:
- project/agent-toolkit
- domain/operations
- domain/testing
- op/runbook
---

# Testing Runbook

## Coverage Baseline

**0% automated test reachability** (confirmed via jcodemunch `get_untested_symbols`, 2026-04-18).

This is intentional: all tools call Tavily and LLM APIs at runtime. Unit tests with mocks would hide real integration failures. The test suite requires live API keys.

---

## Test Suite Location

`agent-toolkit/tests/` — integration tests only (no unit tests, no mocks).

---

## Running Tests

```bash
cd agent-toolkit

# Set required environment variables
export TAVILY_API_KEY=tvly-...
export ANTHROPIC_API_KEY=sk-ant-...
# Or: export OPENAI_API_KEY=sk-...  (if using OpenAI models)

# Install with dev dependencies
pip install -e ".[dev]"

# Run all tests
pytest tests/

# Run with verbose output
pytest tests/ -v

# Run a specific test file
pytest tests/test_tools.py -v
```

---

## Environment Variables Required

| Variable | Purpose | Required |
|----------|---------|---------|
| `TAVILY_API_KEY` | Tavily API calls (all tools) | Always |
| `ANTHROPIC_API_KEY` | Claude models (default config) | For tests using Claude |
| `OPENAI_API_KEY` | OpenAI models | If ModelConfig uses OpenAI |

Tests will fail with auth errors if keys are missing — not skip.

---

## What to Test When Adding a Tool

When adding a new tool:

1. Test the **happy path**: valid inputs → expected output shape
2. Test **API response handling**: malformed/empty Tavily response
3. Test **parameter coercion** if applicable (e.g., string→list like `tavily_search`)
4. Test **observability output**: verify `TavilyUsage` and `LLMUsage` are populated if tool tracks usage

---

## Coverage Gap Registry

| Symbol | File | Gap Type | Notes |
|--------|------|----------|-------|
| All 6 tools | `tools/` | No unit tests | Integration-only by design |
| `ainvoke_with_fallback` | `utilities/utils.py` | No mock-based unit test | Fallback logic untestable without mocking LLM |
| `generate_subqueries` | `utilities/utils.py` | Integration only | Requires real LLM call |
| `HybridResearcher` | `agents/hybrid_researcher.py` | Integration only | Requires Tavily + LLM |
| Use-case scripts | `use-cases/` | No tests | Example programs, not library code |

---

## CI Integration

```yaml
# .github/workflows/test.yml
# Triggered on: PR and push to main
# Requires: TAVILY_API_KEY and ANTHROPIC_API_KEY repository secrets
```

See `.github/workflows/test.yml` for the full CI test configuration.

---

## Philosophy

From `CONTRIBUTING.md` and architectural decisions:
- Integration tests > unit tests for this package — the value is in real API behavior
- Mock-based tests would give false confidence for a library whose purpose is API integration
- 0% jcodemunch test reachability is a feature, not a bug: all paths are exercised against live systems

---

## Relations

- Related: [[agent-toolkit — Architecture Index]]
- Related: [[agent-toolkit — Operations Runbooks Index]]
- Related: [[ADR-000 — agent-toolkit Init]]

---

## Evolution Log

| Date | Change | Trigger |
|------|--------|---------|
| 2026-04-18 | Created; coverage baseline established as 0% (by design) | Phase 3-T5 ingestion via get_untested_symbols |