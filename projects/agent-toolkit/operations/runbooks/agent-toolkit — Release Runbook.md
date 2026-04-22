---
title: agent-toolkit — Release Runbook
type: reference
permalink: agent-kb/projects/agent-toolkit/operations/runbooks/agent-toolkit-release-runbook
created: 2026-04-18
modified: 2026-04-18
entity_type: reference
status: evergreen
tags:
- project/agent-toolkit
- domain/operations
- domain/release
- op/runbook
---

# Release Runbook

## Source

`.github/workflows/publish.yml`

## Overview

Fully automated PyPI release pipeline. Publishing is triggered by bumping the version in `agent-toolkit/pyproject.toml` and pushing to `main`. The workflow reads the version, checks for duplicates on both git tags and PyPI, builds, publishes, and creates a git tag — all idempotently.

---

## Trigger

```yaml
on:
  push:
    branches: [main]
    paths:
      - 'agent-toolkit/pyproject.toml'
  workflow_dispatch:  # Manual trigger from GitHub UI
```

**Rule**: Only pushes to `main` that touch `agent-toolkit/pyproject.toml` trigger the pipeline. No tag-based trigger needed — the workflow creates the tag.

---

## Pipeline Steps

```
1. Checkout (fetch-depth: 0 — full tag history needed)
2. Read version from pyproject.toml via tomllib
3. Check git tag existence (v{version}) → skip if exists
4. Check PyPI for version existence via curl → skip if exists
5. pip install build
6. python -m build  (from agent-toolkit/ dir)
7. pypa/gh-action-pypi-publish (PYPI_API_TOKEN secret)
8. git tag -a "v{version}" + push
```

Steps 5–7 are **gated** by `if: tag_check == false && pypi_check == false` — idempotency guard.
Step 8 tags regardless of PyPI check (in case tag is missing but version is already on PyPI).

---

## Versioning Procedure

To release a new version:

1. Edit `agent-toolkit/pyproject.toml` — bump `version = "X.Y.Z"` under `[project]`
2. Commit and push to `main`
3. GitHub Actions triggers automatically

**No manual tagging required.** The workflow creates `vX.Y.Z` tag after publish.

---

## Authentication

| Secret | Purpose |
|--------|---------|
| `PYPI_API_TOKEN` | PyPI trusted publisher token (GitHub repository secret) |
| GitHub Actions token | Auto-provided; needs `id-token: write` + `contents: write` permissions |

`id-token: write` enables OIDC-based trusted publishing. `contents: write` needed to push tags.

---

## Idempotency Guards

Two independent checks prevent double-publishing:

1. **Git tag check**: `git rev-parse "v{version}"` — if tag exists, skip build+publish
2. **PyPI check**: `curl pypi.org/pypi/tavily-agent-toolkit/{version}/json` — if 200 response with `"version"` key, skip

Workflow will NOT fail if version already exists — it silently skips. Tag creation step runs independently and still creates the tag if missing.

---

## Build Artifacts

```
agent-toolkit/dist/
├── tavily_agent_toolkit-{version}.tar.gz   (sdist)
└── tavily_agent_toolkit-{version}-py3-none-any.whl  (wheel)
```

Built with `python -m build` using the `build` package. `pypa/gh-action-pypi-publish` uploads from `agent-toolkit/dist/`.

---

## Package Name

PyPI: `tavily-agent-toolkit`
Import: `tavily_agent_toolkit`

---

## Manual Release (Emergency)

If CI is unavailable:

```bash
cd agent-toolkit
pip install build twine
python -m build
twine upload dist/* --username __token__ --password $PYPI_API_TOKEN

# Then tag manually
git tag -a "v{version}" -m "Release v{version}"
git push origin "v{version}"
```

---

## Relations

- Related: [[agent-toolkit — Architecture Index]]
- Related: [[agent-toolkit — Planning Roadmap]]

---

## Evolution Log

| Date | Change | Trigger |
|------|--------|---------|
| 2026-04-18 | Created from .github/workflows/publish.yml | Phase 3-T5 ingestion |