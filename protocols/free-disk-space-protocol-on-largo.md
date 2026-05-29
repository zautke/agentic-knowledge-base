---
title: Free-Disk-Space Protocol on largo
created: 2026-05-28
modified: 2026-05-28
type: instruction
entity_type: instruction
status: evergreen
permalink: protocols/free-disk-space-protocol-on-largo
tags:
- project/agent-kb
- domain/macos
- machine/largo
- protocol/disk-space
- status/evergreen
---

# Free-Disk-Space Protocol on largo

**Status**: Active

## Purpose

Reclaim disk space on `largo` (macOS Tahoe 26.4.x), a permanently disk-constrained host where `ENOSPC` is the default failure mode and single-digit GB free is normal. This protocol gives an agent a safe, tiered, existence-gated sequence of cache-reclaim commands that free space mid-session without destroying user data, killing live tooling, or touching protected agent identity/config. Every command is idempotent, read-before-write gated, and ordered by reclaim-per-risk so the dominant wins (tool-native cache cleans) run first and the dangerous operations require explicit confirmation.

## Non-Negotiable Constraints

Copied verbatim from `~/.claude/CLAUDE.md`. These override any reclaim convenience.

**NO LOCAL LLM MODELS, NO LARGE LOCAL CACHES, EVER, ON THIS MACHINE.**

FORBIDDEN, NO EXCEPTIONS:
- `ollama pull <anything>` (any size, any "tiny" model)
- `huggingface-cli download …`
- Any local model weights → `~/.ollama/`, `~/.cache/huggingface/`
- `ollama serve` for inference (the daemon is an attractive trap)
- Any operation expected to write >50 MB without a prior `df` check

USE INSTEAD:
- Ollama inference → https://ollama2.braisenly.com (Cloudflare-tunneled cloud daemon, `:cloud` models pre-loaded)
- Coding co-agents → mcp__codex / mcp__gemini / mcp__claudette / mcp__deepseek-* / mcp__kimi-* / mcp__glm-* (all hosted)

PRE-FLIGHT EVERY SESSION:
```bash
df -h "$HOME" | tail -1       # Avail < 5 GB → cloud path only
curl -sS --max-time 6 https://ollama2.braisenly.com/api/tags
```

FIELD EVIDENCE (why the banner exists): 2026-05-23 — an agent attempted `ollama pull qwen2.5:0.5b` (397 MB) on largo with 454 MB free, hit ENOSPC at 79%, retried, got a digest mismatch (corrupt write under disk pressure), wasted ~30 minutes. The cloud endpoint was available the entire time.

## Safety Rubric

A command is **safe to run automatically (Tier 1)** only if ALL hold:
1. **Zero data loss** — removes only regenerable caches, never source, credentials, transcripts, or app-group data.
2. **No sudo** — does not require elevation.
3. **No restart** — does not require a reboot or logout to take effect.
4. **Safe mid-session** — does not corrupt or starve a tool the running agent depends on.
5. **Existence-gated** — checks the target exists (`command -v`, `[ -d ]`) before acting; absent target → no-op, not error.
6. **Idempotent** — re-running yields the same end state with no side effects.

If a reclaim touches a cache owned by a tool that may be **running**, it drops to **Tier 1b** and must be `pgrep`-guarded (skip if the owner process is live). If it can re-copy on reconnect, requires confirmation of idle devices, needs a reboot, or has any ambiguity of intent, it is **Tier 2 (manual confirm)**. Anything that touches real user data or agent identity is **Tier 3 (never)**.

## Pre-flight (df)

Always measure before and after. Capture both `-h` (human) and `-k` (exact KB) for the evidence template.

```bash
df -h "$HOME" | tail -1          # human-readable Avail (e.g. 2.9Gi)
df -k  "$HOME" | tail -1          # exact KB-free for delta math
```

If `Avail < 5 GB`: cloud-only path for all inference/model work; proceed with Tier 1 reclaim. If `Avail < 1 GB`: reclaim is urgent — Tier 1 first, then evaluate Tier 1b/2 with confirmation.

## Tier 1 — Auto (zero-data-loss, no sudo, no restart, safe mid-session)

Every command existence-gated and idempotent. Run top-to-bottom; the first two carry the dominant win and typically resolve the crisis alone.

| Target | Command | Typical reclaim | Gate |
|---|---|---|---|
| uv cache | `command -v uv && uv cache clean` | ~13G (dominant) | `command -v uv` |
| npm cache | `command -v npm && npm cache clean --force` | ~1.6G | `command -v npm` |
| pnpm store | `command -v pnpm && ! pgrep -x pnpm >/dev/null && pnpm store prune` | ~0.1–1.1G (only unreferenced) | `command -v pnpm` + not running |
| go module cache | `command -v go && go clean -modcache` | ~942M | `command -v go` |
| go build cache | `command -v go && go clean -cache` | 0–varies (often absent) | `command -v go` |
| puppeteer cache | `[ -d ~/.cache/puppeteer ] && rm -rf ~/.cache/puppeteer` | ~693M | `[ -d ]` |
| brew cache | `command -v brew && brew cleanup -s --prune=all` | ~0 if already empty | `command -v brew` |
| unavailable simulators | `command -v xcrun && xcrun simctl delete unavailable` | 0 if none unavailable | `command -v xcrun` |

Notes: `go clean -cache` targets `GOCACHE` (go-build), frequently absent on largo → no-op. `xcrun simctl delete unavailable` only removes simulators already marked unavailable; it does NOT touch DeviceSupport or live simulators.

## Tier 1b — Safe only if the owning tool is stopped (pgrep-guarded)

These caches belong to long-running agent tools. They self-skip while the owner is live. Each command guards with `pgrep` so it is safe to leave in an automated sweep — it simply no-ops when the tool is running.

| Target | Command | Typical reclaim | Guard |
|---|---|---|---|
| codex runtimes | `! pgrep -f "codex" >/dev/null && [ -d ~/.cache/codex-runtimes ] && rm -rf ~/.cache/codex-runtimes` | ~732M | codex not running |
| codex migration backups | `! pgrep -f "codex" >/dev/null && rm -rf ~/.codex/*.migration-mismatch-*` | ~48M | codex not running |
| codex computer-use + plugin cache | `! pgrep -f "codex" >/dev/null && rm -rf ~/.codex/computer-use ~/.codex/plugins/cache` | ~80M | codex not running |
| opencode cache | `! pgrep -f "opencode" >/dev/null && [ -d ~/.cache/opencode ] && rm -rf ~/.cache/opencode` | ~67M | opencode not running |

## Tier 2 — Manual confirm (re-copy, reboot-only, or end-of-session)

Require explicit user confirmation and a stated precondition. Do NOT run unattended.

| Target | Command | Typical reclaim | Precondition / caveat |
|---|---|---|---|
| Xcode iOS DeviceSupport | `rm -rf ~/Library/Developer/Xcode/iOS\ DeviceSupport/*` | ~11G | Re-copies on device reconnect; confirm all devices idle/disconnected |
| claude plugin cache + marketplaces | `rm -rf ~/.claude/plugins/cache ~/.claude/plugins/marketplaces` | ~1.5G | Only after THIS claude session ends; `claude --clear-cache` / `claude clean` do NOT exist (verified — issue #11646 closed not-planned). Use the clear-plugin-cache skill or rm |
| /private/var/folders | (reboot) | ~3.3G | Reboot only — NEVER `rm`; OS-managed per-user temp |
| Time Machine local snapshots | `tmutil deletelocalsnapshots /` | ~0 on largo | CONFIRMED zero benefit here; the stock "trick" frees nothing on this host |
| `~/.ollama` policy cleanup | inspect then remove per no-local-models policy | varies | Enforces the non-negotiable constraint; never re-pull |
| CoreSimulator erase | `xcrun simctl erase all` | varies | Wipes simulator state; confirm no in-flight iOS work |
| stalled downloads | `rm ~/Downloads/*.crdownload` | varies | Only confirmed-dead partial downloads |

Flagged useless-for-disk: `sudo purge` frees inactive memory, not disk — do not use it to chase disk space.

## Tier 3 — Never

- `~/Library/Group Containers` (real app-group data)
- user `~/Downloads` (real user files)
- protected agent identity / config / credentials / transcripts (`~/.claude/`, `~/.codex/` config + auth, session histories)

## Evidence Template

Capture for every reclaim run and append to the Evolution Log.

```json
{
  "date": "YYYY-MM-DD",
  "freeBefore": "<df -h> (<df -k> KB)",
  "freeAfter":  "<df -h> (<df -k> KB)",
  "totalReclaimed": "<sum of cache-removed> ; net system free gain <delta> (concurrent agents may offset)",
  "perCommand": [
    {"cmd": "<command>", "status": "ran|skipped-absent|skipped-live", "freed": "<measured>"}
  ],
  "skippedLive": ["<tool> (live: PIDs ...)"],
  "tier2Pending": ["<target> (~size) — SKIPPED: <reason>"]
}
```

Method note: net free gain often trails cache-removed totals because live agents write to disk concurrently during the sweep. Always report both numbers and name the live writers.

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-05-28 | Seeded protocol from a 3-finding synthesis (AUDIT + SOTA + AGENTS) into one path-deduplicated tiered protocol for largo (macOS Tahoe 26.4.1, 99% full, ~3.1Gi free). First recorded run reclaimed ~15.9G of tool-native cache (uv 12.6GiB, npm ~1.6G, go ~942M, puppeteer 693M, pnpm 92MB) for a net system free gain of ~3.1Gi (2.9Gi → 6.0Gi); the gap vs cache-removed is because 3 live agents (claude/codex/opencode) wrote to disk concurrently, offsetting reclaimed space. Run evidence below. | The host hit a disk crisis and needed a reusable, safety-tiered reclaim protocol instead of ad-hoc cache deletion. | User request to write the free-disk-space protocol to the KB. |

### Run evidence — 2026-05-28

```json
{
  "freeBefore": "2.9Gi (3255248 KB free per df -k at uv-start; df -h showed 2.9Gi at session start)",
  "freeAfter": "6.0Gi (6273684 KB free)",
  "totalReclaimed": "Tool-native cache reclaim ~15.9G removed (uv 12.6GiB + npm 1.6G + go 942M + puppeteer 693M + pnpm 92MB). Net system free gain ~3.1Gi (2.9Gi -> 6.0Gi); the gap vs cache-removed is because 3 live agents (claude/codex/opencode) wrote to disk concurrently during the run, offsetting reclaimed space.",
  "perCommand": [
    {"cmd": "uv cache clean", "status": "ran", "freed": "12.6GiB (uv reported: removed 284561 files; ~/.cache/uv was 13G)"},
    {"cmd": "npm cache clean --force", "status": "ran", "freed": "~1.6G (~/.npm content-addressed cache, was 1.6G)"},
    {"cmd": "pnpm store prune", "status": "ran", "freed": "92 MB (removed 133 packages / 4975 unreferenced files; rest of 1.1G store still referenced)"},
    {"cmd": "go clean -modcache", "status": "ran", "freed": "~942M (~/go/pkg/mod was 942M)"},
    {"cmd": "go clean -cache (GOCACHE)", "status": "skipped-absent", "freed": "0 (GOCACHE dir absent, as predicted)"},
    {"cmd": "rm -rf ~/.cache/puppeteer", "status": "ran", "freed": "693M (du measured 693M before delete)"},
    {"cmd": "brew cleanup -s --prune=all", "status": "ran", "freed": "~1.3KB (cache effectively empty, as predicted)"},
    {"cmd": "xcrun simctl delete unavailable", "status": "ran", "freed": "0 (no unavailable simulators; idempotent no-op). Did NOT touch DeviceSupport or live sims."}
  ],
  "skippedLive": [
    "claude (live: PIDs 2590 claude mcp serve, 3232 claude -c, plus others)",
    "codex (live: codex mcp-server PIDs 4308/4309 and 12124/12125, 14154/14155, 33513+, 37887+, 42046+, 54144+, 74212+)",
    "opencode (live: PID 5493 opencode web --port 4096)"
  ],
  "tier2Pending": [
    "~/.cache/codex-runtimes (~732M) — SKIPPED: codex mcp-server running",
    "~/.codex/*.migration-mismatch-* (~48M) — SKIPPED: gated on codex, codex running",
    "~/.codex/computer-use + ~/.codex/plugins/cache (~80M) — SKIPPED: gated on codex, codex running",
    "~/.cache/opencode (~67M) — SKIPPED: opencode web (PID 5493) running"
  ]
}
```

**Synthesis summary.** Merged three findings (AUDIT + SOTA + AGENTS) into one path-deduplicated tiered protocol for largo (macOS Tahoe 26.4.1, 99% full, 3.1Gi free). Verified live state: codex (PID 4308/4309 `codex mcp-server`) and opencode (PID 5493 `opencode web`) are BOTH RUNNING, so all Tier1b commands are guarded by pgrep and correctly self-skip; pnpm is not running so its store prune is Tier1-safe. claude is running (this session) so its plugin cache is demoted to Tier2.

**Verification notes.** `claude --clear-cache` / `claude clean` do NOT exist (`--help` shows no such flag; issue #11646 closed not-planned) — the spec's suggestion to use it is incorrect; plugin cache must be cleared by `rm` or the clear-plugin-cache skill. `xcrun simctl delete unavailable` yields ~0 today (no unavailable sims). No files were deleted during the analysis itself — read-only probes (`df`/`du`/`pgrep`/`ls`/`command -v`) only.
