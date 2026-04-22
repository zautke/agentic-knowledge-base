---
title: dot-agents Git Workflow
type: reference
permalink: agent-kb/projects/dot-agents/dot-agents-git-workflow
created: 2026-04-18
modified: 2026-04-18
entity_type: reference
status: evergreen
tags:
- project/agent-kb
- domain/git
- domain/dot-agents
- op/protocol
---

# dot-agents Git Workflow

Rules for the `~/.agents` repository to prevent future git corruption and config drift.

## Branch Model

- **`main`** — single stable branch. Always deployable. Never rebase on main.
- Feature/experimental work: create a short-lived `dev` or `feat/<name>` branch, merge via PR.
- No `backup/*` branches — use git tags instead if preservation is needed.

## Commit Message Format

```
<type>(<scope>): <what>
```

Types: `feat`, `fix`, `chore`, `refactor`  
Scopes: `mcp`, `opencode`, `claude`, `gemini`, `codex`, `hooks`, `skills`, `agents`, `git`

Examples:
- `feat(mcp): add jcodemunch server to all configs`
- `fix(opencode): replace hardcoded PAT with env var reference`
- `chore(skills): add create-skill and using-jcodemunch`

## Rules

### Rule 1 — MCP changes go to `mcp-master.md` first
Add to `settings/global/mcp-master.md` → update all assistant configs → single commit.

### Rule 2 — Never edit assistant configs outside `~/.agents/`
- OpenCode: edit `settings/global/opencode.jsonc` → copy to `~/.config/opencode/opencode.jsonc`
- Claude Code: already symlinked via `dot-agents init`
- Gemini/Codex: edit their files in `settings/global/`

### Rule 3 — dot-agents CLI safeguards
```bash
dot-agents doctor                     # always first
git -C ~/.agents diff                 # verify changes after any dot-agents command
git -C ~/.agents add <specific files> # never git add . or git add -A
dot-agents sync commit                # preferred commit method
```

### Rule 4 — No interactive rebase on main
If git rebase is needed on main, do it on a branch and merge. If a rebase gets stuck:
```bash
git rebase --abort    # clear stale rebase-merge state (may need to remove untracked files first)
```

### Rule 5 — No force push without explicit review
Force push only to fix broken history. Always use `--force-with-lease`, never `--force`.

## Emergency Recovery (if git breaks again)

```bash
# 1. Clear stale rebase state
git rebase --abort
# If that fails due to untracked files:
mv platform/auggie/.auggie.json /tmp/  # move conflicting untracked file
git rebase --abort
mv /tmp/.auggie.json platform/auggie/

# 2. Identify working baseline
git log --oneline --all    # find last known-good commit

# 3. Soft-reset and recommit cleanly
git reset --soft <good-commit>
git add <files>
git commit -m "chore: recover from broken git state"
git push --force-with-lease origin main
```

## History (as of 2026-04-18)

The repo had a complex git incident:
- An `interactive rebase` on `main` was started, got stuck on `fd6e305`, and was abandoned
- `HEAD` somehow ended up on `dev` while `.git/rebase-merge/` still pointed to `main`
- The `d69f0b9 "Initial dot-agents configuration"` commit was inserted out of order (Jan 2026 date but positioned after March commits), causing further confusion
- **Resolution:** `git rebase --abort` → hard-reset main to dev's content (`38250eb`) → soft-reset to baseline (`fd6e305`) → clean recommit → force-push

**Root cause:** `dot-agents` CLI does NOT corrupt git. The corruption was from manual `git rebase -i` operations.

## Relations
- Implements [[dot-agents CLI — Agentic Usage Guide]]
- Related to [[MCP Master List — dot-agents]]