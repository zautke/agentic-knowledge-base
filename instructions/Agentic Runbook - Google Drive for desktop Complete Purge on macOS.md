---
title: Agentic Runbook - Google Drive for desktop Complete Purge on macOS
type: instruction
permalink: instructions/agentic-runbook-google-drive-for-desktop-complete-purge-on-mac-os
created: 2026-03-11
modified: 2026-03-11
entity_type: instruction
status: evergreen
tags:
- project/agent-kb
- domain/macos
- domain/google-drive
- status/evergreen
- op/create-node
---

# Agentic Runbook - Google Drive for desktop Complete Purge on macOS

## Purpose

This runbook defines the exact agent workflow for fully removing Google Drive for desktop from a Mac, including the app bundle, DriveFS data, logs, relaunch hooks, and residual user-library state that commonly survives a normal uninstall.

Use this when the goal is one of:
- recover from broken Drive sync state
- recover from repeated sign-in/config corruption
- prepare for a clean reinstall
- remove Drive before deprovisioning or ownership transfer
- eliminate stale DriveFS state during troubleshooting

## Authoritative Basis
Primary sources verified on 2026-03-12:
- Google Admin Help: https://support.google.com/a/answer/7582171?hl=en
- Google Drive Help, macOS / File Provider behavior: https://support.google.com/drive/answer/12178485?hl=en
- Google Drive Help, troubleshooting and disconnect-account warnings: https://support.google.com/drive/answer/2565956?hl=en
- Google Drive Help, advanced guide and cache/offline behavior: https://support.google.com/drive/answer/16631477?hl=en
- Google Workspace Admin Help, release notes: https://support.google.com/a/answer/7577057?hl=en
- Apple Support: https://support.apple.com/en-us/102610
- Apple File Provider domain-removal semantics (Context7 surfaced): https://developer.apple.com/documentation/FileProvider/NSFileProviderManager/DomainRemovalMode

Google explicitly documents these macOS Drive for desktop data locations:
- `~/Library/Application Support/Google/DriveFS/`
- `~/Library/Logs/Google/DriveFS/`

Everything beyond those two directories should be treated as operational cleanup and machine-verified, not as a Google-canonical uninstall matrix. The 2026-03-12 additions for Containers, Group Containers, Application Scripts, and preserved `CloudStorage` domains are based on local verification plus Apple File Provider behavior, not on a published Google uninstall matrix.
## Agent Operating Rules

1. Do not begin deletion until Drive is confirmed stopped.
2. Capture pre-state evidence before removing files.
3. Prefer narrow, path-explicit deletion commands.
4. Treat non-Google-documented plist/cache/launch-agent paths as verify-before-delete targets.
5. Reboot or log out/in before declaring success.
6. Record exact commands used and final verification results.

## Inputs Required

- local shell access on the Mac
- permission to remove the app from `/Applications`
- confirmation whether this is:
  - full removal only
  - clean reinstall prep
  - troubleshooting reset preserving unrelated Google apps

## Definition of Done

The purge is complete only when all of the following are true:
1. No `Google Drive` or `DriveFS` process remains running.
2. `Google Drive.app` is removed from `/Applications`.
3. `~/Library/Application Support/Google/DriveFS/` does not exist.
4. `~/Library/Logs/Google/DriveFS/` does not exist.
5. No Drive-specific Container, Group Container, or Application Scripts residue remains if the mission is literal zero-trace removal.
6. No Drive-specific `~/Library/CloudStorage/GoogleDrive*` domain remains unless it was intentionally archived for data retention.
7. No login item or launch agent respawns Drive.
8. Post-reboot verification remains clean.

## Execution Flow

### Phase 0: Preflight and Evidence Capture

Record current process, app, and file state:

```bash
pgrep -ifl 'Google Drive|DriveFS' || true
ls -ld '/Applications/Google Drive.app' 2>/dev/null || true
ls -ld ~/Library/Application\ Support/Google/DriveFS 2>/dev/null || true
ls -ld ~/Library/Logs/Google/DriveFS 2>/dev/null || true
ls -1 ~/Library/LaunchAgents 2>/dev/null | rg 'drivefs|google' || true
```

Capture whether Google Drive is configured as a login item:
- System Settings -> General -> Login Items
- Record any Google Drive / DriveFS related entry before removal

### Phase 1: Stop the Running Client

First, quit Drive through the menu bar UI if present.

Then enforce process shutdown:

```bash
pkill -TERM -f 'Google Drive|DriveFS' || true
sleep 2
pgrep -ifl 'Google Drive|DriveFS' || true
pkill -KILL -f 'Google Drive|DriveFS' || true
pgrep -ifl 'Google Drive|DriveFS' || true
```

Acceptance condition:
- final `pgrep` returns no Drive-related process

### Phase 2: Remove the App Bundle

```bash
mv '/Applications/Google Drive.app' ~/.Trash/ 2>/dev/null || true
```

Acceptance condition:

```bash
test ! -e '/Applications/Google Drive.app' && echo APP_REMOVED
```

### Phase 3: Remove Google-Documented DriveFS Data

These are the highest-confidence purge targets because Google documents them directly.

```bash
rm -rf ~/Library/Application\ Support/Google/DriveFS
rm -rf ~/Library/Logs/Google/DriveFS
```

Acceptance condition:

```bash
test ! -e ~/Library/Application\ Support/Google/DriveFS && echo DRIVEFS_SUPPORT_REMOVED
test ! -e ~/Library/Logs/Google/DriveFS && echo DRIVEFS_LOGS_REMOVED
```

### Phase 4: Remove Ancillary User-Level State

These paths are operational cleanup targets. Verify existence before deletion.

Inspect:

```bash
ls -l ~/Library/Preferences/com.google.drivefs.settings.plist 2>/dev/null || true
ls -l ~/Library/Preferences/com.google.drivefs.plist 2>/dev/null || true
ls -d ~/Library/Caches/com.google.drivefs* 2>/dev/null || true
ls -l ~/Library/LaunchAgents/com.google.drivefs.*.plist 2>/dev/null || true
ls -d ~/Library/Saved\ Application\ State/com.google.drivefs.savedState 2>/dev/null || true
```

Delete if present:

```bash
rm -f ~/Library/Preferences/com.google.drivefs.settings.plist
rm -f ~/Library/Preferences/com.google.drivefs.plist
rm -f ~/Library/LaunchAgents/com.google.drivefs.*.plist
rm -rf ~/Library/Caches/com.google.drivefs*
rm -rf ~/Library/Saved\ Application\ State/com.google.drivefs.savedState
```

### Phase 4A: Remove App Extension Containers and Shared Container State

If the objective is a literal zero-trace purge rather than only a DriveFS reset, inspect and remove the Drive-specific container residue that may survive the earlier phases:

```bash
ls -ld ~/Library/Group\ Containers/EQHXZ8M8AV.group.com.google.drivefs 2>/dev/null || true
ls -ld ~/Library/Containers/com.google.drivefs.fpext 2>/dev/null || true
ls -ld ~/Library/Containers/com.google.drivefs.finderhelper 2>/dev/null || true
ls -ld ~/Library/Containers/com.google.drivefs.finderhelper.findersync 2>/dev/null || true
ls -ld ~/Library/Application\ Scripts/EQHXZ8M8AV.group.com.google.drivefs 2>/dev/null || true
ls -ld ~/Library/Application\ Scripts/com.google.drivefs.fpext 2>/dev/null || true
ls -ld ~/Library/Application\ Scripts/com.google.drivefs.finderhelper 2>/dev/null || true
ls -ld ~/Library/Application\ Scripts/com.google.drivefs.finderhelper.findersync 2>/dev/null || true
```

Delete if present and if the mission is true complete removal:

```bash
rm -rf ~/Library/Group\ Containers/EQHXZ8M8AV.group.com.google.drivefs
rm -rf ~/Library/Containers/com.google.drivefs.fpext
rm -rf ~/Library/Containers/com.google.drivefs.finderhelper
rm -rf ~/Library/Containers/com.google.drivefs.finderhelper.findersync
rm -rf ~/Library/Application\ Scripts/EQHXZ8M8AV.group.com.google.drivefs
rm -rf ~/Library/Application\ Scripts/com.google.drivefs.fpext
rm -rf ~/Library/Application\ Scripts/com.google.drivefs.finderhelper
rm -rf ~/Library/Application\ Scripts/com.google.drivefs.finderhelper.findersync
```

### Phase 4B: Inspect File Provider CloudStorage Remnants After Backup

Before deleting any `~/Library/CloudStorage/GoogleDrive*` path, copy out any user data that must survive. Google’s current troubleshooting and advanced-guide docs warn that disconnect and cache-clearing steps can destroy offline-only or unsynced data.

Inspect:

```bash
ls -ld ~/Library/CloudStorage/GoogleDrive* 2>/dev/null || true
find ~/Library/CloudStorage -maxdepth 1 -name 'GoogleDrive*' -print 2>/dev/null || true
```

If the mission is literal complete removal and the required data has been preserved elsewhere, remove the Drive-specific CloudStorage domains too:

```bash
rm -rf ~/Library/CloudStorage/GoogleDrive*
```

Special caution:
A preserved-domain marker such as `.drive_fs_ignore_preserved_domain` indicates that macOS/File Provider likely preserved prior Google Drive data intentionally. Treat this as data-bearing state, not disposable junk, until you have verified backup/retention requirements.

### Phase 5: Remove Relaunch Hooks

Check login items manually:
- System Settings -> General -> Login Items
- Remove any Google Drive item still present

Then verify no launch agent remains:

```bash
ls -1 ~/Library/LaunchAgents 2>/dev/null | rg 'drivefs|google' || true
```

Acceptance condition:
- no Drive-specific launch agent remains
- no Drive login item remains

### Phase 6: Reboot Boundary

Reboot or log out/in. Do not skip this if the objective is a true clean purge.

Reason:
- clears residual background state
- prevents false success from an already-loaded user session
- validates that no startup hook recreates Drive

### Phase 7: Post-Reboot Verification

Run:

```bash
pgrep -ifl 'Google Drive|DriveFS' || true
ls -ld ~/Library/Application\ Support/Google/DriveFS 2>/dev/null || true
ls -ld ~/Library/Logs/Google/DriveFS 2>/dev/null || true
ls -1 ~/Library/LaunchAgents 2>/dev/null | rg 'drivefs|google' || true
```

Expected clean state:
- no running Drive-related process
- no DriveFS support directory
- no DriveFS logs directory
- no Drive relaunch artifact under LaunchAgents
- no login item in System Settings

## 2026-04-09 Execution Addendum: Finder and TCC Edge Cases

Machine-verified on local host `luke` during an executed purge on macOS `26.3` with active Drive version `123.0.1.0`.

### Observed execution realities
- direct shell removal of `/Applications/Google Drive.app` did not succeed even after process shutdown because the app bundle was root-owned and regular shell deletion paths were insufficient in the current session
- Finder-mediated deletion succeeded for the installed app bundle and also succeeded for the same bundle after it had been moved into `~/.Trash`
- `~/Library/Application Support/Google/DriveFS` briefly reappeared during execution while Finder Sync or related app components were still active, then stayed absent after the app bundle was fully removed and final cleanup reran
- `~/Library/Containers/com.google.drivefs.fpext`
- `~/Library/Containers/com.google.drivefs.finderhelper`
- `~/Library/Containers/com.google.drivefs.finderhelper.findersync`
  remained present as tiny residues and returned `Operation not permitted` under `rm -rf` even after process shutdown and app removal

### Operational consequence
For clean reinstall prep, treat the purge as successful when all of the following are true even if the three protected container directories remain:
- app bundle absent
- DriveFS support and log directories absent
- no active Drive-related process remains
- retained `~/Library/CloudStorage/GoogleDrive*` domains are intentionally preserved or explicitly handled

For literal zero-trace removal, expect an extra manual step or stronger privacy/TCC access to remove the protected container directories.

### Preferred operator behavior
1. stop Drive processes
2. if shell removal of `/Applications/Google Drive.app` fails, use Finder-mediated delete before concluding the app-removal phase
3. rerun DriveFS directory verification after app removal because partial components may recreate state during the same session
4. if the three `~/Library/Containers/com.google.drivefs.*` paths remain but are only tiny metadata residues and all higher-value state is gone, record them explicitly instead of falsely claiming total removal

## One-Pass Command Sequence

Use only after preflight confirms this machine should be fully purged:

```bash
pkill -TERM -f 'Google Drive|DriveFS' || true
sleep 2
pkill -KILL -f 'Google Drive|DriveFS' || true
mv '/Applications/Google Drive.app' ~/.Trash/ 2>/dev/null || true
rm -rf ~/Library/Application\ Support/Google/DriveFS
rm -rf ~/Library/Logs/Google/DriveFS
rm -f ~/Library/Preferences/com.google.drivefs.settings.plist
rm -f ~/Library/Preferences/com.google.drivefs.plist
rm -f ~/Library/LaunchAgents/com.google.drivefs.*.plist
rm -rf ~/Library/Caches/com.google.drivefs*
rm -rf ~/Library/Saved\ Application\ State/com.google.drivefs.savedState
```

## Failure Modes and Handling

### Drive process will not stay stopped
- Re-run process checks after `pkill -TERM`
- Use `pkill -KILL` only after confirming the graceful stop failed
- Check login items and launch agents before declaring the purge blocked

### App bundle removal fails
- Confirm the process is fully stopped
- Check whether `/Applications/Google Drive.app` still has open file handles or requires elevation

### DriveFS directories reappear after reboot
- Check login items again
- Check `~/Library/LaunchAgents/`
- Confirm a reinstall agent or MDM policy is not re-deploying Drive

### User wants reinstall after purge
Do not reinstall until post-reboot verification is clean.

## Evidence Template

Record the following in any operational log:
- date/time of purge
- hostname / machine identity
- reason for purge
- whether reinstall is intended
- preflight results
- paths found and removed
- whether login items existed
- whether reboot occurred
- post-reboot verification output

## Anti-Patterns

| Anti-pattern | Why it is wrong | Correct behavior |
|---|---|---|
| Removing files before stopping Drive | Can leave partial state or recreate files mid-purge | Stop processes first, then delete |
| Treating app deletion as complete uninstall | Leaves DriveFS state and logs behind | Always purge DriveFS support/log paths |
| Pretending all plist/cache paths are officially documented | Overstates certainty | Mark ancillary paths as machine-verify cleanup |
| Declaring success before reboot | Relaunch hooks may still recreate state | Reboot or log out/in, then re-check |

## Relations
- related_to [[Google Drive for desktop Complete Purge on macOS]]
- related_to [[Fundamental – Agent KB]]
- related_to [[Action Taxonomy – Index]]

## Evolution Log

| Date | Change | Reason | Trigger |
|---|---|---|---|
| 2026-03-11 | Created agentic execution runbook for macOS Google Drive for desktop purge | Convert verified procedural note into repeatable operational workflow with guardrails and acceptance checks | User requested an agentic runbook |
| 2026-04-09 | Added Finder-mediated app deletion and protected container residue guidance from executed local purge on `luke` | Future operators need explicit handling for root-owned app bundles, state rehydration during same-session purge, and TCC-protected container remnants | User requested live purge execution and KB curation of the outcome |

## Automation Reference

Repository script:
- `scripts/google-drive-purge-macos.sh`

Safe default:
```bash
scripts/google-drive-purge-macos.sh
```

Structured evidence JSON:
```bash
scripts/google-drive-purge-macos.sh --json > purge-report.json
```

Human-readable log capture:
```bash
scripts/google-drive-purge-macos.sh --log-file /tmp/google-drive-purge.log
```

Actual execution with evidence capture:
```bash
scripts/google-drive-purge-macos.sh --execute --json > purge-report.json
scripts/google-drive-purge-macos.sh --execute --log-file /tmp/google-drive-purge.log
```

Behavior notes:
- `--dry-run` is the default
- `--json` emits a structured preflight/postflight report to stdout while human logs move to stderr
- `--log-file PATH` appends human-readable execution output to the specified path
- `--keep-app` preserves `/Applications/Google Drive.app` while still purging user-level state

## Evolution Log Addendum

- 2026-03-11: Added repository automation reference for `scripts/google-drive-purge-macos.sh`, including `--json` and `--log-file` evidence modes.
- 2026-03-12: Added zero-trace purge phases for Drive-specific Containers, Group Containers, Application Scripts, and preserved `CloudStorage` File Provider domains. Also added explicit backup caution for offline-only and unsynced data based on current Google docs and local verification of an active `122.0` install.
