---
title: Instruction Manual - Google Drive for desktop Recovery Operations on macOS
type: instruction
permalink: instructions/instruction-manual-google-drive-for-desktop-recovery-operations-on-mac-os
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
- source/largo
- machine/largo
---

# Instruction Manual - Google Drive for desktop Recovery Operations on macOS

## Agent Curation Metaprompt (Adapted)
ROLE: You are the operations trainer and recovery curator for Google Drive for desktop on macOS. This manual is a living operational artifact for agents and trainees who need to diagnose, purge, repair, and cleanly reinstall Drive for desktop without losing procedural rigor.

GOVERNING PRINCIPLES:
1. ACCUMULATE, NEVER SUMMARIZE AWAY. Preserve concrete paths, commands, failure signatures, and environmental conditions.
2. STRUCTURED INCREMENTAL UPDATES ONLY. Add or refine named sections; do not rewrite the whole manual casually.
3. EVIDENCE-LINKED ENTRIES. Every new failure mode or guardrail requires date, context, example, and source.
4. SELF-REFERENTIAL VERIFICATION. Check this manual and linked runbooks before adding new guidance.
5. EVOLUTION LOG IS MANDATORY. Append every meaningful operational change.
6. TRIGGER CONDITIONS FOR UPDATE. Update this manual when: a new DriveFS/File Provider failure mode appears, Google changes supported macOS/File Provider behavior, purge/reinstall steps change, disk-space/cache behavior changes, or an operator discovers a missing verification step.
7. WHAT NOT TO CHANGE. Do not remove failure ledgers or anti-pattern entries; deprecate them with reason.
8. QUALITY SIGNAL TRACKING. Record which verification steps were decisive and which sections prevented operator error.

## Purpose and Scope

This manual teaches a trainee or agent how to operate Google Drive for desktop recovery on macOS as a disciplined workflow rather than an ad hoc uninstall.

Use it for:
- DriveFS corruption or stale sync state
- repeated sign-in or startup failures
- File Provider mount issues on macOS 12.1+
- clean purge before reinstall
- maintenance and prevention of future corruption

Out of scope:
- Windows or Linux Drive workflows
- enterprise MDM packaging details
- account-side Google Admin policy changes

## Source Backbone
Primary sources verified 2026-03-12:
- Google Admin Help, Drive for desktop log collection and path reference: https://support.google.com/a/answer/7582171?hl=en
- Google Drive for desktop setup/install: https://support.google.com/drive/answer/10838124?hl=en
- Google Drive for desktop compatibility: https://support.google.com/drive/answer/2375082?hl=en
- Google Drive for desktop on macOS / File Provider behavior: https://support.google.com/drive/answer/12178485?hl=en
- Google Drive for desktop troubleshooting: https://support.google.com/drive/answer/2565956?hl=en
- Google Drive for desktop release notes: https://support.google.com/a/answer/7577057?hl=en
- Google advanced/settings guidance for cache-location and related limits: https://support.google.com/drive/answer/16631477?hl=en
- Google settings/customization page referenced by Google search results for cache and disk-space behavior: https://support.google.com/drive/answer/13470231?hl=en
- Apple standard uninstall guidance: https://support.apple.com/en-us/102610
- Apple File Provider domain-removal semantics (Context7 surfaced): https://developer.apple.com/documentation/FileProvider/NSFileProviderManager/DomainRemovalMode
## High-Value Background the Trainee Must Understand

### 1. What Drive for desktop actually is on macOS

Drive for desktop is not just an app bundle. It is a coordinated system composed of:
- the app: `Google Drive.app`
- DriveFS user support data under `~/Library/Application Support/Google/DriveFS/`
- DriveFS logs under `~/Library/Logs/Google/DriveFS/`
- background processes such as `Google Drive`, `crashpad_handler`, Finder helper, and File Provider extension processes
- Finder integration and, on supported modern macOS, File Provider-backed cloud storage behavior

Operational lesson:
Removing only `/Applications/Google Drive.app` is not a clean reset.

### 2. File Provider changes the runtime model

Google documents that on macOS 12.1 and later, streaming uses Apple File Provider. That changes where files appear and how recovery behaves:
- File Provider path for advanced users: `~/Library/CloudStorage`
- Legacy streaming path: `/Volumes/GoogleDrive`
- Finder-side enablement may be required under `Locations`
- Downloaded/local files can remain accessible even when Drive is not running, depending on File Provider state

Operational lesson:
A trainee must determine whether the machine is using File Provider semantics before assuming legacy mount behavior.

### 3. The most important directories and what they mean

| Path | Purpose | Recovery significance |
|---|---|---|
| `/Applications/Google Drive.app` | App bundle | Remove during purge unless preserving app for state-only reset |
| `~/Library/Application Support/Google/DriveFS/` | Core user-level DriveFS support data, caches, Crashpad DB, state | Primary purge target |
| `~/Library/Logs/Google/DriveFS/` | DriveFS logs | Primary purge target and best evidence source |
| `~/Library/Preferences/com.google.drivefs.settings.plist` | User preferences/settings | Secondary cleanup target |
| `~/Library/Preferences/com.google.drivefs.plist` | Additional preference state | Secondary cleanup target |
| `~/Library/LaunchAgents/com.google.drivefs.*.plist` | Relaunch hooks | Remove if present during cleanup |
| `~/Library/Caches/com.google.drivefs*` | Cache remnants | Secondary cleanup target |
| `~/Library/Saved Application State/com.google.drivefs.savedState` | Saved app state | Secondary cleanup target |
| `~/Library/Group Containers/EQHXZ8M8AV.group.com.google.drivefs` | Shared Drive/File Provider state used by app extensions | Zero-trace cleanup target discovered during 2026-03-12 local verification |
| `~/Library/Containers/com.google.drivefs.fpext` | File Provider extension container | Zero-trace cleanup target discovered during 2026-03-12 local verification |
| `~/Library/Containers/com.google.drivefs.finderhelper` | Finder helper container | Zero-trace cleanup target discovered during 2026-03-12 local verification |
| `~/Library/Containers/com.google.drivefs.finderhelper.findersync` | Finder Sync extension container | Zero-trace cleanup target discovered during 2026-03-12 local verification |
| `~/Library/Application Scripts/com.google.drivefs.*` | App-extension script support directories | Zero-trace cleanup target discovered during 2026-03-12 local verification |
| `~/Library/CloudStorage` | File Provider cloud mount location on modern macOS | Inspect for active or preserved `GoogleDrive*` domains before reinstall or before claiming zero-trace cleanup |
| `/Volumes/GoogleDrive` | Legacy streaming path | Relevant only on older/non-File Provider workflows |

## Canonical Recovery Workflow

### Phase 0: Classify the incident before touching files

Ask and answer:
1. Is this a pure uninstall, a corruption reset, or a clean reinstall prep?
2. Is the machine on macOS 12.1+?
3. Is the observed behavior File Provider-based (`~/Library/CloudStorage`) or legacy mount-based (`/Volumes/GoogleDrive`)?
4. Is disk pressure part of the incident?
5. Is the user asking only for a dry run or for actual deletion?

### Phase 1: Evidence capture

Before any destructive action, capture:

```bash
sw_vers -productVersion
ps -axo pid=,comm= | rg 'Google Drive|DriveFS|FinderSyncExtension|DFSFileProviderExtension' || true
ls -ld '/Applications/Google Drive.app' 2>/dev/null || true
ls -ld ~/Library/Application\ Support/Google/DriveFS 2>/dev/null || true
ls -ld ~/Library/Logs/Google/DriveFS 2>/dev/null || true
df -h ~
```

Why this matters:
- version decides File Provider expectations
- process list confirms active moving parts
- directory checks confirm what state exists
- free-space check tests the disk-pressure hypothesis

### Phase 2: Dry-run planning first

Use the repository script as the first operational move:

```bash
scripts/google-drive-purge-macos.sh --dry-run --json > purge-report.json
```

This gives:
- exact targets that would be removed
- preflight process state
- launch-agent indicators
- a machine-readable artifact for review

Operator rule:
Do not execute deletion until the dry-run output has been reviewed and approved.

### Phase 3: Controlled purge

The purge sequence is:
1. stop Drive-related processes
2. remove `Google Drive.app` unless intentionally preserving it
3. remove DriveFS support data
4. remove logs
5. remove ancillary preferences/caches/launch agents if present
6. remove login-item relaunch hooks manually in System Settings
7. reboot or log out/in
8. verify the state stayed clean after reboot

Use:
```bash
scripts/google-drive-purge-macos.sh --execute --json > purge-report.json
scripts/google-drive-purge-macos.sh --execute --log-file /tmp/google-drive-purge.log
```

### Phase 4: Clean reinstall

After verification is clean:
1. confirm supported OS version
2. download the current installer from Google
3. install the app
4. launch and sign in
5. grant Privacy and Security permissions if requested
6. verify Finder-side Google Drive / File Provider enablement
7. verify mount state and process state

## Contingency Planning and Self-Healing

### Failure Mode A: Drive processes will not stay stopped

Symptoms:
- `Google Drive` or File Provider helper processes immediately return after `pkill`

Likely causes:
- login item still enabled
- launch agent still present
- partial app state still loaded into current login session

Mitigation:
1. re-check login items in `System Settings -> General -> Login Items`
2. inspect `~/Library/LaunchAgents`
3. reboot after removing relaunch artifacts
4. do not trust same-session cleanliness if processes keep returning

### Failure Mode B: App deleted, corruption persists after reinstall

Symptoms:
- reinstall succeeds but sign-in loops, sync errors, or stale state returns

Likely causes:
- DriveFS support data not removed
- cache/preferences not removed
- reboot boundary skipped

Mitigation:
1. verify `~/Library/Application Support/Google/DriveFS/` is truly absent before reinstall
2. verify logs directory is absent
3. verify post-reboot clean state before reinstalling

### Failure Mode C: File Provider mount does not appear cleanly

Symptoms:
- Drive installs but Finder `Locations` does not show Google Drive correctly
- mount exists but is not enabled or accessible

Likely causes:
- Finder-side enablement not approved
- privacy/permission prompt skipped
- macOS/File Provider initialization issue

Mitigation:
1. open Finder and inspect `Locations`
2. click `Google Drive` and approve `Enable` if shown
3. restart Drive
4. if still failing, reboot and retest
5. ensure the OS is current enough and compatible

### Failure Mode D: Low disk space or cache starvation corrupts runtime behavior

Symptoms:
- sync stalls or uploads fail unexpectedly
- cache does not update correctly
- state appears stale or inconsistent after routine operation

Important operational finding from this session:
- the triggering suspicion was exhaustion of physical disk space preventing the cache from updating properly

Google-linked guidance indicates:
- Drive’s cache location and behavior matter
- uploads can fail if the source folder is larger than available space in the partition where the cache lives
- on macOS File Provider mode, some legacy cache-location controls differ from the older streaming model

Mitigation:
1. always inspect `df -h ~` during diagnosis
2. maintain a free-space floor before large syncs or reindexing events
3. if disk pressure is high, resolve storage pressure before blaming auth or reinstall alone
4. after reclaiming space, restart Drive and retest before escalating to full purge

### Failure Mode E: Operator executes deletion too early

Symptoms:
- evidence is lost
- user did not approve destructive actions
- postmortem quality drops

Mitigation:
1. dry-run first, always
2. collect JSON and log artifacts first
3. explicitly wait for approval before `--execute`

### Failure Mode F: Preserved File Provider domains and app-extension containers survive purge

Symptoms:
- `DriveFS` support and log directories are gone, but Drive-specific `~/Library/CloudStorage/GoogleDrive*` folders remain
- File Provider or Finder integration behaves as if an older domain is still present
- Drive-specific containers under `~/Library/Containers`, `~/Library/Group Containers`, or `~/Library/Application Scripts` remain after the standard purge

Likely causes:
- macOS/File Provider preserved previously downloaded or dirty user data
- the purge removed only the documented DriveFS paths, not extension/container residue
- the operator treated `CloudStorage` remnants as disposable without first classifying whether they still contain user data

Mitigation:
1. inspect `~/Library/CloudStorage/GoogleDrive*` explicitly before reinstall
2. treat preserved-domain markers such as `.drive_fs_ignore_preserved_domain` as evidence of retained File Provider state
3. copy out any data that must survive before deleting the old `CloudStorage` domain
4. remove Drive-specific Containers, Group Containers, and Application Scripts if the mission is literal zero-trace cleanup
5. reboot and verify that no Drive-specific plugin or process remains before reinstalling

### Failure Mode G: App bundle removal requires Finder even after process shutdown

Symptoms:
- shell removal or move of `/Applications/Google Drive.app` fails or appears to succeed incompletely
- the app bundle remains present after `mv` or `rm` attempts
- the app bundle may become stuck as a root-owned item inside `~/.Trash`

Local verified example on 2026-04-09:
- machine: `luke`
- macOS: `26.3`
- direct shell deletion did not complete the app-removal phase
- Finder-mediated delete succeeded both for the installed app and for the trashed bundle

Likely causes:
- ownership or permission boundaries on the app bundle
- macOS or Finder handling of app bundles differs from plain shell file removal in the current user session

Mitigation:
1. confirm processes are stopped first
2. if shell removal fails, switch to Finder-mediated delete instead of repeating blind `rm -rf`
3. if the app lands in `~/.Trash` but remains undeletable from the shell, delete it through Finder

### Failure Mode H: Protected File Provider container directories survive an otherwise successful purge

Symptoms:
- `DriveFS` support and log directories are gone
- the app bundle is gone
- no Drive-related process remains
- but `~/Library/Containers/com.google.drivefs.fpext`, `~/Library/Containers/com.google.drivefs.finderhelper`, or `~/Library/Containers/com.google.drivefs.finderhelper.findersync` remain
- shell deletion returns `Operation not permitted`

Local verified example on 2026-04-09:
- machine: `luke`
- the three directories remained as tiny residues after the purge
- the higher-value DriveFS and group-container state was already gone

Likely causes:
- macOS container-manager or TCC protections on app-extension container directories
- residue is metadata-level state rather than meaningful retained DriveFS payload

Mitigation:
1. treat this as a distinct residual-state class rather than proof that the full purge failed
2. record the remaining directories explicitly in the evidence log
3. for clean reinstall prep, proceed after reboot if app bundle, DriveFS support/logs, and active processes are gone
4. for literal zero-trace removal, use a stronger manual or privacy-approved deletion path after the main purge

## Maintenance of a Clean Reinstall

### Root causes that commonly destabilize Drive

1. Disk exhaustion / cache starvation
2. Interrupted updates or background process churn
3. File Provider enablement or permission gaps
4. stale login items / relaunch hooks after partial removal
5. reinstalling before purge verification is clean

### Proactive guardrails

#### Guardrail 1: Keep headroom on the home volume

Minimum practical rule:
- do not let the home volume drift into chronic low-space conditions
- inspect `df -h ~` whenever Drive problems appear
- before large sync operations, confirm healthy free space on the partition that hosts the relevant cache/runtime state

#### Guardrail 2: Treat DriveFS and File Provider as stateful subsystems

Do not assume restarting the UI alone is enough. When troubleshooting:
- inspect processes
- inspect DriveFS directories
- inspect Finder/File Provider behavior

#### Guardrail 3: Use evidence artifacts for every purge

Always capture one or both:
- JSON report: `scripts/google-drive-purge-macos.sh --dry-run --json > purge-report.json`
- log file: `scripts/google-drive-purge-macos.sh --dry-run --log-file /tmp/google-drive-purge.log`

#### Guardrail 4: Separate diagnosis from destruction

The safe order is:
1. diagnose
2. dry-run
3. approval
4. execute
5. reboot
6. verify
7. reinstall only if necessary

#### Guardrail 5: Validate the post-reinstall environment

After reinstall:
- verify sign-in
- verify Finder integration
- verify expected File Provider path behavior
- verify no immediate sync/startup error

#### Guardrail 6: Preserve offline-only and unsynced data before destructive cleanup

Current Google docs add two critical data-loss warnings that must change operator behavior:
- disconnecting an account can delete `Lost & Found`
- in streaming mode, unsynced local changes can be lost if cache is cleared or corrupted

Operational rule:
Before account disconnect, cache-clearing, or deleting any `~/Library/CloudStorage/GoogleDrive*` domain, copy out any offline-only, mirrored, or unsynced user data that must survive the purge.

## Anti-Pattern Registry

| Anti-pattern | Impact | Corrective action |
|---|---|---|
| Deleting `Google Drive.app` and calling it a full uninstall | Leaves DriveFS corruption behind | Purge support/log paths too |
| Skipping dry run | Surprises user and loses evidence | Run dry-run first and save artifacts |
| Reinstalling before rebooted clean verification | Re-imports stale runtime conditions | Verify clean state first |
| Ignoring disk space during diagnosis | Misses a primary corruption trigger | Check `df -h ~` early |
| Assuming `/Volumes/GoogleDrive` on modern File Provider machines | Wrong mount expectations | Determine File Provider vs legacy mode first |

## Quality Signals

Track after each use:
- Did the dry-run artifact change the execution decision?
- Was disk-space inspection decisive?
- Did the reboot boundary catch a relaunch condition?
- Did the operator need the File Provider section to resolve confusion?

## Relations
- related_to [[Google Drive for desktop Complete Purge on macOS]]
- related_to [[Agentic Runbook - Google Drive for desktop Complete Purge on macOS]]
- related_to [[Agentic Runbook - Google Drive for desktop Clean Reinstall on macOS]]
- related_to [[Fundamental – Agent KB]]
- related_to [[Action Taxonomy – Index]]

## Evolution Log
| Date | Change | Reason | Trigger |
|---|---|---|---|
| 2026-03-11 | Created comprehensive instructional manual for macOS Google Drive recovery, including File Provider background, contingency planning, and maintenance guardrails | Convert session learnings and verified sources into a trainer-grade manual for future agents/operators | User requested an operation instruction manual with contingency and maintenance planning |
| 2026-03-12 | Added zero-trace cleanup targets for Drive-specific Containers, Group Containers, Application Scripts, preserved `CloudStorage` domains, and explicit data-loss guardrails for offline-only and unsynced content | Capture new official-doc findings plus local verification of residue that survives the original purge model | Official-doc refresh and local inspection of an active `122.0` install revealed missing cleanup and backup steps |
| 2026-04-09 | Added Finder-required app deletion and TCC-protected container residue failure modes from executed local purge on `luke` | Preserve machine-verified behavior that changes how operators should interpret shell deletion failures and tiny residual container state | User requested live purge execution and KB curation of the result |
