---
title: Google Drive for desktop Complete Purge on macOS
type: note
permalink: operations/macos/google-drive-for-desktop-complete-purge-on-mac-os
tags:
- google-drive
- macos
- uninstall
- purge
- operations
---

# Google Drive for desktop Complete Purge on macOS

## Purpose

Document the current, verified procedure to fully remove Google Drive for desktop from a Mac, including the app, user-level support data, logs, preferences, and relaunch hooks that can cause the client to reappear or retain stale state.

## Scope

This procedure is for **Google Drive for desktop** on **macOS**. It is intended for cases where a normal uninstall is not enough, including:
- broken sync state
- repeated login/config corruption
- stale DriveFS cache or logs
- migration to a clean reinstall
- full local removal before deprovisioning a machine

## Sources
Primary sources verified on 2026-03-12:
- Google Drive for desktop log collection and data path reference: https://support.google.com/a/answer/7582171?hl=en
- Google Drive for desktop on macOS / File Provider behavior: https://support.google.com/drive/answer/12178485?hl=en
- Google Drive for desktop troubleshooting: https://support.google.com/drive/answer/2565956?hl=en
- Google Drive for desktop release notes: https://support.google.com/a/answer/7577057?hl=en
- Manage Google Drive for desktop: Advanced guide: https://support.google.com/drive/answer/16631477?hl=en
- Apple standard app uninstall guidance for macOS: https://support.apple.com/en-us/102610
- Apple File Provider domain-removal semantics (Context7 surfaced): https://developer.apple.com/documentation/FileProvider/NSFileProviderManager/DomainRemovalMode
## Key Facts from Sources

- Google documents these user-level macOS locations for Drive for desktop support data:
  - `~/Library/Application Support/Google/DriveFS/`
  - `~/Library/Logs/Google/DriveFS/`
- Apple’s standard uninstall guidance for Mac apps is to quit the app and move it to the Trash; supporting files under the user Library are not automatically guaranteed to be removed.

## Procedure

### 1. Quit Google Drive for desktop completely

From the Google Drive menu bar item, quit Drive for desktop.

Then verify no active process remains:

```bash
pgrep -ifl 'Google Drive|DriveFS'
```

If processes remain, terminate them cleanly first, then force if needed:

```bash
pkill -TERM -f 'Google Drive|DriveFS'
sleep 2
pkill -KILL -f 'Google Drive|DriveFS'
```

### 2. Remove the application bundle

Use standard macOS app removal:
- Finder -> `Applications` -> move `Google Drive.app` to Trash

Or via shell:

```bash
mv '/Applications/Google Drive.app' ~/.Trash/
```

### 3. Remove Drive for desktop support data

Delete the documented Google Drive for desktop support-data directories:

```bash
rm -rf ~/Library/Application\ Support/Google/DriveFS
rm -rf ~/Library/Logs/Google/DriveFS
```

These are the most important purge targets because Google explicitly documents them as the Drive for desktop macOS data/log locations.

### 4. Remove user preference and relaunch artifacts

Google’s support page does not enumerate every ancillary macOS preference/plist path. In practice, these are the next locations to inspect and remove if present:

```bash
rm -f ~/Library/Preferences/com.google.drivefs.settings.plist
rm -f ~/Library/Preferences/com.google.drivefs.plist
rm -f ~/Library/LaunchAgents/com.google.drivefs.*.plist
rm -rf ~/Library/Caches/com.google.drivefs*
rm -rf ~/Library/Saved\ Application\ State/com.google.drivefs.savedState
```

Because Google does not currently publish all of these as canonical removal paths, treat them as operational cleanup targets to verify on the specific machine before deletion.

### 5. Remove login-item / background restart hooks

After deleting files, inspect for restart hooks:
- System Settings -> General -> Login Items
- Remove any Google Drive / DriveFS related entry if still present

Also verify no launch agent remains under:

```bash
ls -1 ~/Library/LaunchAgents | rg 'drivefs|google'
```

### 6. Reboot or log out/in

A reboot is recommended after purge to clear any stuck file-provider or background-agent state.

### 7. Verify the purge

After reboot, confirm all of the following:

```bash
pgrep -ifl 'Google Drive|DriveFS'
ls ~/Library/Application\ Support/Google/DriveFS
ls ~/Library/Logs/Google/DriveFS
```

Expected result:
- no running Drive/DriveFS processes
- the DriveFS support/log paths do not exist
- no active login item or launch agent recreates the process

## Fast Path Script

Use only after verifying the machine should be fully purged:

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

## Reinstall Note

If the objective is a clean reinstall, reinstall only after:
- reboot/log out-in is complete
- `DriveFS` paths remain absent
- no login item respawns Google Drive automatically
- any preserved File Provider `~/Library/CloudStorage/GoogleDrive*` domains have either been archived intentionally or removed intentionally

## 2026-03-12 Addendum: File Provider and Container Residue

New material verified on 2026-03-12 from official Google/Apple docs plus local inspection of an active version `122.0` install:

- Google’s latest release note on 2026-03-02 lists `122.0` for macOS and Windows, so a corrupted `122.0` install is not automatically an outdated-client incident.
- Google’s current troubleshooting guidance warns that disconnecting an account can delete `Lost & Found`, and the advanced guide warns that unsynced changes in streaming mode can be lost if local cache is cleared or corrupted.
- Apple’s uninstall guidance says to prefer an app-provided uninstaller if present. Local inspection of `/Applications/Google Drive.app/Contents` on 2026-03-12 found `diagnostic_tool` but no obvious bundled uninstaller, so manual purge remains the operative removal path for this install.
- Local inspection on 2026-03-12 found additional Drive-specific residue not covered by the original note or by the repository purge script:
  - `~/Library/Group Containers/EQHXZ8M8AV.group.com.google.drivefs`
  - `~/Library/Containers/com.google.drivefs.fpext`
  - `~/Library/Containers/com.google.drivefs.finderhelper`
  - `~/Library/Containers/com.google.drivefs.finderhelper.findersync`
  - `~/Library/Application Scripts/EQHXZ8M8AV.group.com.google.drivefs`
  - `~/Library/Application Scripts/com.google.drivefs.fpext`
  - `~/Library/Application Scripts/com.google.drivefs.finderhelper`
  - `~/Library/Application Scripts/com.google.drivefs.finderhelper.findersync`
- Local inspection also found multiple `~/Library/CloudStorage/GoogleDrive*` domains, including one preserved-domain folder containing `.drive_fs_ignore_preserved_domain`. This is consistent with Apple File Provider domain-removal behavior that can preserve downloaded or dirty user data.

Operational consequence:
If the goal is literally "remove all traces," the purge must inspect and usually remove Drive-specific Containers, Group Containers, Application Scripts, and preserved `CloudStorage` domains after any required user data has been copied out.

## 2026-04-09 Execution Addendum: Local `luke` Purge Outcome

Machine-verified outcome from an executed purge on host `luke`.

### Verified removals
- `/Applications/Google Drive.app` was ultimately removed, but Finder-mediated deletion was required
- `~/Library/Application Support/Google/DriveFS` was removed
- `~/Library/Logs/Google/DriveFS` was removed
- `~/Library/Group Containers/EQHXZ8M8AV.group.com.google.drivefs` was removed
- Drive-related Application Scripts were removed

### Preserved by design
- `~/Library/CloudStorage/GoogleDrive*` was preserved because the session goal was clean reinstall prep rather than literal zero-trace deletion of retained File Provider domains

### Residue that remained
The following protected container directories remained and returned `Operation not permitted` under shell deletion even after app removal and process shutdown:
- `~/Library/Containers/com.google.drivefs.fpext`
- `~/Library/Containers/com.google.drivefs.finderhelper`
- `~/Library/Containers/com.google.drivefs.finderhelper.findersync`

These were tiny residues rather than the high-value DriveFS payload.

### Operational consequence
The note should distinguish two success states:
- clean reinstall prep: app absent, DriveFS support/logs absent, no active processes, CloudStorage intentionally preserved, tiny protected container residues may remain
- literal zero-trace purge: all of the above plus explicit removal of retained `CloudStorage` domains and any protected container residues

## Confidence and Gaps

### High confidence
- `~/Library/Application Support/Google/DriveFS/`
- `~/Library/Logs/Google/DriveFS/`
- quit app first, then remove app bundle
- preserve or export any unsynced/offline-only data before cache-clearing or account-disconnect actions

### Moderate confidence / machine-verify
- preference, cache, saved-state, and launch-agent cleanup paths beyond the two Google-documented DriveFS directories
- Drive-specific Containers, Group Containers, and Application Scripts discovered during 2026-03-12 local inspection
- preserved `~/Library/CloudStorage/GoogleDrive*` domains that may represent intentionally retained File Provider data rather than pure junk

Reason: Google’s public support material explicitly documents the DriveFS support/log paths and File Provider behavior, but not a full forensic uninstall matrix for every ancillary macOS user-library artifact. The container and preserved-domain findings are evidence-backed local verification items that align with Apple File Provider semantics.

## Evolution Log

- 2026-03-11: Created from current-source verification using Google Admin Help and Apple Support. Marked non-Google-published ancillary paths as operational cleanup targets rather than canonical documented paths.
- 2026-03-12: Added File Provider preserved-domain and app-container residue findings, plus data-loss cautions from current Google troubleshooting and advanced-guide docs. Trigger: official-doc refresh and local verification of an active `122.0` install.
- 2026-04-09: Added executed-purge findings from local host `luke`, including Finder-mediated app deletion and protected container residue that survived shell deletion. Trigger: user-requested live execution of the purge protocol.
