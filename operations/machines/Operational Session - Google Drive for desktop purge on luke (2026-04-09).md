---
title: Operational Session - Google Drive for desktop purge on luke (2026-04-09)
type: note
permalink: operations/machines/operational-session-google-drive-for-desktop-purge-on-luke-2026-04-09
created: 2026-04-09
modified: 2026-04-09
entity_type: note
status: active
tags:
- project/agent-kb
- domain/macos
- domain/google-drive
- status/active
---

# Operational Session - Google Drive for desktop purge on luke (2026-04-09)

## User Request
Execute the Google Drive corruption purge protocols from Basic Memory on the local Mac and preserve enough evidence to support follow-on reinstall work.

## Incident Classification
- `purge+reinstall`
- exact execution mode used: clean reinstall prep while preserving `~/Library/CloudStorage/GoogleDrive*`

## Initial State
- machine: `luke`
- macOS version: `26.3`
- free space before purge was critically low and the data volume was effectively full during preflight
- active install observed: `Google Drive` / `DriveFS` version `123.0.1.0`
- active processes during preflight included `Google Drive`, `crashpad_handler`, and `FinderSyncExtension`
- `/Applications/Google Drive.app` existed and was root-owned
- `~/Library/Application Support/Google/DriveFS` existed
- Drive-specific File Provider residue existed:
  - `~/Library/Group Containers/EQHXZ8M8AV.group.com.google.drivefs`
  - `~/Library/Containers/com.google.drivefs.fpext`
  - `~/Library/Containers/com.google.drivefs.finderhelper`
  - `~/Library/Containers/com.google.drivefs.finderhelper.findersync`
  - `~/Library/Application Scripts/EQHXZ8M8AV.group.com.google.drivefs`
  - `~/Library/Application Scripts/com.google.drivefs.fpext`
  - `~/Library/Application Scripts/com.google.drivefs.finderhelper`
  - `~/Library/Application Scripts/com.google.drivefs.finderhelper.findersync`
- preserved File Provider domains existed:
  - `~/Library/CloudStorage/GoogleDrive-lukezautke@gmail.com`
  - `~/Library/CloudStorage/GoogleDrive-lukezautke@gmail.com (1-9-26 9:48 PM)`

## Execution Summary
- stopped active Google Drive and Finder Sync processes
- moved `/Applications/Google Drive.app` to Trash via Finder because direct shell move/removal was blocked by ownership/permissions
- deleted the trashed app via Finder because the root-owned app bundle remained permission-protected inside `~/.Trash`
- removed `~/Library/Application Support/Google/DriveFS`
- removed `~/Library/Logs/Google/DriveFS`
- removed `~/Library/Group Containers/EQHXZ8M8AV.group.com.google.drivefs`
- removed Drive-related Application Scripts except for paths that were briefly rehydrated before final cleanup
- preserved `~/Library/CloudStorage/GoogleDrive*` intentionally to avoid destructive removal of retained File Provider data during reinstall prep

## Final State
- `/Applications/Google Drive.app`: absent
- `~/.Trash/Google Drive.app`: absent
- `~/Library/Application Support/Google/DriveFS`: absent in final check
- `~/Library/Logs/Google/DriveFS`: absent in final check
- active Drive-related processes: none observed in final check
- free space after purge remained critically low: `/dev/disk3s1   228Gi   193Gi   1.5Gi   100%`

## Remaining Residue
The following tiny protected container directories remained because macOS returned `Operation not permitted` even after process shutdown and app removal:
- `~/Library/Containers/com.google.drivefs.fpext`
- `~/Library/Containers/com.google.drivefs.finderhelper`
- `~/Library/Containers/com.google.drivefs.finderhelper.findersync`

Operational interpretation:
- these residues were tiny metadata-level remnants rather than the high-value DriveFS payload
- the purge reached a practical clean-reinstall-prep state, but not literal zero-trace removal

## Lessons Learned
- shell-driven purge can accidentally self-match if broad `pkill -f` patterns are executed inside a command string containing the same pattern; PID-targeted shutdown is safer for agent execution wrappers
- on this machine, root-owned app bundles can survive a shell move/delete path but Finder-mediated delete succeeds under the logged-in admin user context
- moving a root-owned app into `~/.Trash` does not guarantee shell deletability afterward; Finder delete may still be required to remove the trashed bundle
- even after Drive is fully stopped and the app is removed, `~/Library/Containers/com.google.drivefs.*` can remain protected by macOS and resist `rm -rf`
- preserving `~/Library/CloudStorage/GoogleDrive*` remains the right default for reinstall prep when the user has not explicitly requested literal zero-trace removal of retained File Provider data

## Evidence
- local summary artifact: `/tmp/google-drive-purge-summary-20260409-145051.txt`

## Relations
- related_to [[Instruction Manual - Google Drive for desktop Recovery Operations on macOS]]
- related_to [[Agentic Runbook - Google Drive for desktop Complete Purge on macOS]]
- related_to [[Google Drive for desktop Complete Purge on macOS]]
- related_to [[Agentic Runbook - Google Drive for desktop Clean Reinstall on macOS]]

## Evolution Log

| Date | Change | Reason | Trigger |
|---|---|---|---|
| 2026-04-09 | Created operational session record for the executed local purge on `luke` | Preserve exact machine-verified findings, blocked deletion cases, and reinstall-prep state in Basic Memory | User requested execution of the purge protocols and then requested KB curation of the outcome |
