---
title: Agentic Runbook - Google Drive for desktop Clean Reinstall on macOS
type: instruction
permalink: instructions/agentic-runbook-google-drive-for-desktop-clean-reinstall-on-mac-os
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

# Agentic Runbook - Google Drive for desktop Clean Reinstall on macOS

## Purpose

This runbook defines the agent workflow for a clean reinstall of Google Drive for desktop on macOS after a verified purge.

Use this when:
- the prior installation was corrupted
- DriveFS state was purged intentionally
- the machine needs a known-clean Drive setup
- sign-in or File Provider integration needs to be re-established from zero

## Authoritative Basis

Primary sources verified on 2026-03-11:
- Google Drive for desktop setup: https://support.google.com/drive/answer/10838124?hl=en
- Google Drive for desktop compatibility: https://support.google.com/drive/answer/2375082?hl=en
- Google Drive for desktop on macOS / File Provider permissions: https://support.google.com/drive/answer/12178485?hl=en

## Key Facts from Current Docs

- macOS compatibility requirement: `macOS Monterey 12.1 and up`
- installer download: `https://dl.google.com/drive-file-stream/GoogleDrive.dmg`
- on macOS 12.1 and up, streaming uses Apple File Provider
- Drive may require Privacy and Security permissions for folders, removable volumes, Photos, and sign-in hardware paths
- Finder-side Google Drive enablement may be required under `Locations`

## Preconditions

Before reinstalling, verify:
1. purge is complete
2. no `Google Drive` or `DriveFS` process is running
3. `~/Library/Application Support/Google/DriveFS/` does not exist
4. `~/Library/Logs/Google/DriveFS/` does not exist
5. no login item is set to relaunch the old instance

## Definition of Done

The reinstall is complete only when all of the following are true:
1. Google Drive for desktop installs successfully
2. user can sign in successfully
3. Drive appears in Finder or menu bar as expected
4. File Provider or mirror/stream mode initializes cleanly
5. no startup or sync error is present after first launch

## Execution Flow

### Phase 0: Verify Environment

Check the macOS version:

```bash
sw_vers -productVersion
```

Acceptance condition:
- version is `12.1` or newer

### Phase 1: Confirm Purge Baseline

Run:

```bash
ps -axo pid=,comm= | rg 'Google Drive|DriveFS|FinderSyncExtension|DFSFileProviderExtension' || true
ls -ld ~/Library/Application\ Support/Google/DriveFS 2>/dev/null || true
ls -ld ~/Library/Logs/Google/DriveFS 2>/dev/null || true
```

Acceptance condition:
- no active Drive-related processes
- no residual DriveFS support or log directory

### Phase 2: Download Installer

Official installer URL:
- `https://dl.google.com/drive-file-stream/GoogleDrive.dmg`

Example:

```bash
curl -L -o ~/Downloads/GoogleDrive.dmg https://dl.google.com/drive-file-stream/GoogleDrive.dmg
```

Acceptance condition:
- `~/Downloads/GoogleDrive.dmg` exists

### Phase 3: Install the App

Open the DMG and follow the installer flow:

```bash
open ~/Downloads/GoogleDrive.dmg
```

Then complete the standard macOS installation steps.

Acceptance condition:
- `/Applications/Google Drive.app` exists

Verify:

```bash
test -e '/Applications/Google Drive.app' && echo APP_INSTALLED
```

### Phase 4: First Launch and Sign-In

Launch the app:

```bash
open -a 'Google Drive'
```

Then sign in through the browser flow when prompted.

Acceptance condition:
- account successfully signs in
- Drive menu appears in the menu bar

### Phase 5: macOS Permissions and Finder Enablement

If Drive requests permissions, grant them in:
- System Settings -> Privacy and Security
- relevant sections include `Files and Folders` and `Photos`

If using File Provider on macOS 12.1+, enable Google Drive in Finder if prompted:
1. Open Finder
2. Under `Locations`, click `Google Drive`
3. Click `Enable` if shown

Acceptance condition:
- Finder access works
- Drive can mount its cloud-backed location successfully

### Phase 6: Select Sync Mode and Validate Mount

If prompted, select the intended sync approach:
- streaming
- mirroring

Note:
- on modern macOS streaming uses File Provider
- advanced users can expect the File Provider path under `~/Library/CloudStorage`

Validate expected mount behavior:
- Finder sidebar shows Google Drive
- menu bar app opens without startup error
- files/folders are visible

### Phase 7: Initial Health Check

Verify the process is running:

```bash
ps -axo pid=,comm= | rg 'Google Drive|DriveFS|FinderSyncExtension|DFSFileProviderExtension' || true
```

If File Provider is used, inspect expected cloud storage path:

```bash
ls -ld ~/Library/CloudStorage 2>/dev/null || true
```

Acceptance condition:
- Drive is running cleanly
- Finder integration is active
- no immediate sync/login/startup error is present

## Failure Modes and Handling

### Installer will not run
- verify macOS version is supported
- re-download the DMG from Google
- confirm the prior install was actually purged

### Sign-in loops or browser auth fails
- confirm old Drive processes are not still running
- confirm the purge baseline was clean before reinstall
- restart the app and retry

### Drive appears but files do not mount
- check Finder `Locations`
- click `Google Drive` and approve `Enable` if shown
- restart Drive or reboot if permissions were just granted

### Desktop/Documents/Photos sync permission errors
- open System Settings -> Privacy and Security
- grant required access under `Files and Folders` or `Photos`
- restart Drive after permission changes

### Startup failure with File Provider
- Google advises updating macOS and restarting the computer

## Evidence Template

Record:
- date/time of reinstall
- hostname / machine identity
- macOS version
- installer source URL used
- whether permissions were requested and granted
- whether File Provider enablement was required
- initial sync mode selected
- post-install validation result

## Anti-Patterns

| Anti-pattern | Why it is wrong | Correct behavior |
|---|---|---|
| Reinstalling before purge verification | Carries stale DriveFS state forward | Verify clean baseline first |
| Ignoring macOS permission prompts | Causes missing folders/devices or broken sync expectations | Grant required permissions and restart Drive if needed |
| Assuming Finder integration is automatic | File Provider enablement may require explicit user approval | Check Finder `Locations` and enable Google Drive |
| Declaring success before first launch validation | Install can succeed while startup/auth still fails | Validate sign-in, mount, and running process state |

## Relations
- related_to [[Agentic Runbook - Google Drive for desktop Complete Purge on macOS]]
- related_to [[Google Drive for desktop Complete Purge on macOS]]
- related_to [[Fundamental – Agent KB]]
- related_to [[Action Taxonomy – Index]]

## Evolution Log

| Date | Change | Reason | Trigger |
|---|---|---|---|
| 2026-03-11 | Created clean reinstall runbook for macOS Google Drive for desktop | Pair the purge workflow with a validated reinstall sequence based on current Google documentation | User requested companion reinstall guidance |