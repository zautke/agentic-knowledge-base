---
title: Ollama Local Service (olsvc.sh) - Operational Session 2026-02-18
type: reference
permalink: operations/ollama/ollama-local-service-olsvc.sh-operational-session-2026-02-18
created: 2026-02-18
modified: 2026-02-18
entity_type: reference
status: evergreen
tags:
- project/agent-kb
- domain/ollama
- domain/operations
- domain/launchd
- asset/script
- op/update-node
---

# Ollama Local Service (olsvc.sh) - Operational Session 2026-02-18

## Purpose and Scope
Operational record for `olsvc.sh` enhancements and runtime audit outcomes captured on 2026-02-18, focused on LaunchAgent control, environment integrity, API operability, and failure diagnostics.

## 1) Baseline Target and Control Plane
- Script path reference: `/Users/luke/.local/bin/olsvc/olsvc.sh`
- Symlink baseline: `/Users/luke/.local/bin/olsvc/olsvc.sh -> ../olsvc.sh`
- LaunchAgent label controlled by script: `com.ollama.local`
- Canonical operational root: `/Volumes/DEV/.ollama`
- Canonical env file path in script: `/Volumes/DEV/.ollama/ollama.env`

## 2) Command Surface Added in This Session
### Newly added command families
- `env`
- `plist`
- `servers`
- `api`

### Command behavior details
- `env show`: prints running server process environment.
- `plist`: prints running server plist in XML.
- `servers`: enumerates localhost Ollama listeners and endpoint URLs.
- `api`: adds comprehensive API subcommands with `jq` pretty-print support across core and compatibility endpoints.

### Reliability hardening added
- Exact launch label matching for safer process targeting.
- Safer log-tail behavior.
- API failure paths now return non-zero exit status.

## 3) Follow-up Enhancements Added in This Session
### `preflight [--takeover]`
- Detects target bind host/port from plist-derived runtime intent.
- Detects active listener conflicts on target port.
- Supports optional `launchctl` takeover for Ollama-labeled conflicts (`--takeover`).

### `audit [lines]`
- Summarizes stderr signal classes:
  - bind conflicts
  - pull stalls
  - TLS handshake timeouts
  - DNS failures
  - runner timeouts
  - context canceled
- Summarizes stdout API status classes and top endpoint/status combinations.

### `env` command extensions
- `env show`
- `env drift [env_file]`: compares canonical env vs plist env vs running env and warns on duplicates.
- `env import [env_file] [--restart]`: strict import with duplicate-key rejection, normalization, plist validation, backup, optional restart.

## 4) Official Documentation Mapping Used for Implementation
Implementation was aligned against official Ollama documentation families:
- Ollama API references (core API behavior and endpoint contract)
- Ollama CLI references
- OpenAI compatibility references
- Anthropic compatibility references

### Mapping outcome
- `api` subcommand design mapped to both core Ollama endpoints and compatibility endpoint families.
- Compatibility-aware command coverage was included rather than only native Ollama endpoints.

## 5) Operational-Plane Audit Findings (`/Volumes/DEV/.ollama`)
### Key findings
- Historical bind conflicts were frequent (`bind: address already in use`).
- Config drift detected in `/Volumes/DEV/.ollama/ollama.env`, including duplicate keys.
- Mixed historical host bindings observed (`0.0.0.0` and `127.0.0.1`).
- Pull/download instability observed (TLS handshake timeouts, stalled parts, EOF/reset).
- API error statuses observed in stdout (`500`, `499`, `401`, `400`, `404`).
- Non-Ollama artifacts found in `.ollama/logs` and repo-like tooling directories under `.ollama`.
- Size signals observed: models approx `27G`; logs approx `29M`.

## 6) Validation Outcomes
- Script syntax checks passed.
- `preflight` validated locally.
- `audit` validated locally.
- `env drift` validated and reported duplicate keys in canonical env file.
- `env import` currently fails by design due to duplicate key rejection in canonical env file.

## 7) Contradiction and Duplication Check
- Pre-creation note search in `agent-kb` found no existing `olsvc`/Ollama operation note to update.
- Runtime evidence shows conflicting historical binding modes (`0.0.0.0` vs `127.0.0.1`); this is retained as an unresolved operational contradiction rather than normalized away.

## 8) Unresolved Risks and Follow-ups
1. Canonical env remediation:
   - Remove duplicate keys in `/Volumes/DEV/.ollama/ollama.env` so `env import` can complete successfully.
2. Port ownership stability:
   - Use `preflight --takeover` policy decisions carefully to avoid taking over non-target services.
3. Download reliability:
   - Continue tracking TLS timeout/stall/reset frequency and correlate to network/DNS conditions.
4. Binding policy normalization:
   - Decide and enforce single binding strategy (`127.0.0.1` only vs explicit LAN exposure) and document rationale.
5. Log hygiene:
   - Separate non-Ollama artifacts from `.ollama/logs` workflow to improve signal quality in `audit` output.

## 9) Evidence Registry
| Date | Context | Concrete example | Source |
|------|---------|------------------|--------|
| 2026-02-18 | Baseline script targeting | `olsvc/olsvc.sh` confirmed as symlink to `../olsvc.sh`; script controls `com.ollama.local` and `/Volumes/DEV/.ollama` paths | Session implementation and local file inspection (`/Users/luke/.local/bin/olsvc/olsvc.sh`) |
| 2026-02-18 | Feature implementation | Added `env`, `plist`, `servers`, `api`; extended `env` with `show/drift/import`; added `preflight` and `audit` | Session code changes in `olsvc.sh` |
| 2026-02-18 | Reliability hardening | Exact launch label matching, safe log-tail behavior, non-zero API failure exits implemented | Session code changes in `olsvc.sh` |
| 2026-02-18 | Operational drift findings | Duplicate env keys, bind conflicts, mixed host binding, API error status classes, pull/download instability identified | Runtime logs and audit outcomes under `/Volumes/DEV/.ollama` |
| 2026-02-18 | Validation result | `env drift` reports duplicate keys and `env import` fails intentionally until duplicates are fixed | Local command validation outcomes in session |

## 10) Quality Signals
- Most useful changes: `preflight` conflict detection and `audit` aggregation materially reduce diagnosis time.
- Protective behavior confirmed: strict duplicate-key rejection in `env import` prevents silent config corruption.
- Current gap: canonical env file quality blocks import/restart automation until cleanup is completed.

## Evolution Log
| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-02-18 | Created operational record for `olsvc.sh` enhancements, audit findings, and validations | Preserve session-critical service operations knowledge with evidence and explicit risks | Curator session capture request for Basic Memory `agent-kb` |

## Relations
- related_to [[Index - Ollama Local Service Operations]]
- related_to [[Machine Profile - largo]]
- related_to [[Index – Agent KB]]