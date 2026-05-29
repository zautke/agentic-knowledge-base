---
title: SSH Recovery Snapshot - largo and adagio (2026-04-28)
type: reference
entity_type: reference
status: active
created: 2026-04-28
modified: 2026-04-28
permalink: operations/machines/ssh-recovery-snapshot-largo-adagio-2026-04-28
tags:
- project/agent-kb
- domain/infrastructure
- domain/networking
- domain/ssh
- domain/windows-openssh
- domain/macos
- asset/recovery-snapshot
- machine/largo
- machine/adagio
---

# SSH Recovery Snapshot - largo and adagio (2026-04-28)

## Purpose
Persist the known-good SSH recovery state for `largo` and `adagio` after repairing:

- `adagio -> ssh largo` username mismatch (`luke` -> `derp`).
- Plain interactive `ssh largo` from `adagio` not loading `largo`'s bash startup.

This snapshot is intended for catastrophic recovery where an agent must rebuild SSH client/server configuration on both machines.

## Snapshot Root
All captured files live under:

`agent-kb/operations/machines/ssh-recovery-snapshots/2026-04-28-largo-adagio/`

Machine splits:

- `largo/`
- `adagio/`

## Security Boundary
Private key material is intentionally not copied into this KB snapshot.

The snapshot includes:

- SSH client configs.
- SSH server configs.
- `authorized_keys` and `authorized_keys2` public authorization files.
- `known_hosts`.
- Public keys.
- Host public keys.
- Inventories with filenames, permissions/metadata, service state, effective `ssh -G` output, and public-key fingerprints.

Recovery still requires private key material from the user's secure key store or key regeneration followed by `authorized_keys` updates.

## Repaired State
### largo
- User: `derp`
- Hostname: `largo.local`
- LAN IP: `10.4.10.13`
- Login shell in Directory Services remains `/bin/zsh`.
- Added `~/.bash_profile` so bash login shells source `~/.bashrc`.

`~/.bash_profile`:

```bash
#!/usr/bin/env bash

if [ -r "$HOME/.bashrc" ]; then
  . "$HOME/.bashrc"
fi
```

### adagio
- User: `adagio\me`
- Hostname: `ADAGIO`
- LAN IP: `10.4.10.15`
- `Host largo` now resolves to `User derp`.
- `Host largo` now forces an interactive bash login on `largo`:

```sshconfig
RequestTTY yes
RemoteCommand /opt/homebrew/bin/bash -l
```

This avoids macOS launching the account's default zsh shell for the nested workflow and ensures `.bash_profile -> .bashrc` is loaded.

## Validation
Validated from `largo`:

```text
ssh adagio
# wait for PowerShell profile load
ssh largo
```

Validated nested shell state on `largo`:

```text
BASH_VERSION=5.3.3(1)-release
BASHRC_HOME=/Users/derp/.bash
repeat is a function
```

The login visibly sourced the bash startup flow:

```text
Sourcing .bashrc.lnx ...
Sourcing .bashrc.macos ...
```

## Captured Files
### largo
- `largo/user-ssh/config`
- `largo/user-ssh/authorized_keys`
- `largo/user-ssh/known_hosts`
- `largo/user-ssh/*.pub`
- `largo/etc-ssh/ssh_config`
- `largo/etc-ssh/sshd_config`
- `largo/etc-ssh/100-macos.conf`
- `largo/etc-ssh/99-terminal-stability.conf`
- `largo/etc-ssh/crypto.conf`
- `largo/homebrew-etc-ssh/ssh_config`
- `largo/homebrew-etc-ssh/sshd_config`
- `largo/bash_profile`
- `largo/bashrc.symlink-target-copy`
- `largo/inventory.txt`

### adagio
- `adagio/user-ssh/config`
- `adagio/user-ssh/ssh_config`
- `adagio/user-ssh/ssh_config.win.programdata.ssh.txt`
- `adagio/user-ssh/authorized_keys`
- `adagio/user-ssh/authorized_keys2`
- `adagio/user-ssh/known_hosts`
- `adagio/user-ssh/*.pub`
- `adagio/programdata-ssh/sshd_config`
- `adagio/programdata-ssh/administrators_authorized_keys`
- `adagio/programdata-ssh/ssh_host_ecdsa_key.pub`
- `adagio/programdata-ssh/ssh_host_ed25519_key.pub`
- `adagio/programdata-ssh/ssh_host_rsa_key.pub`
- `adagio/inventory.txt`

## Restore Notes
### largo
1. Recreate `/Users/derp/.ssh` with mode `700`.
2. Restore `largo/user-ssh/config` to `/Users/derp/.ssh/config`.
3. Restore `authorized_keys`, `known_hosts`, and public keys from `largo/user-ssh/`.
4. Restore `/Users/derp/.bash_profile` from `largo/bash_profile`.
5. Restore `/etc/ssh` server/client configs from `largo/etc-ssh/` when using Apple OpenSSH.
6. Restore `/opt/homebrew/etc/ssh` configs from `largo/homebrew-etc-ssh/` when using Homebrew OpenSSH.
7. Reinstall private keys from secure storage or regenerate and update peer `authorized_keys`.
8. Verify with `ssh -o BatchMode=yes largo true` locally and then from `adagio`.

### adagio
1. Recreate `C:\Users\me\.ssh`.
2. Restore user SSH configs/public authorization files from `adagio/user-ssh/`.
3. Restore `C:\ProgramData\ssh\sshd_config` and public host keys from `adagio/programdata-ssh/`.
4. Restore `C:\ProgramData\ssh\administrators_authorized_keys` if administrator-class SSH login is required.
5. Reinstall private keys from secure storage or regenerate and update peer `authorized_keys`.
6. Restart Windows OpenSSH:

```powershell
Restart-Service sshd
Start-Service ssh-agent
```

7. Verify:

```powershell
ssh -G largo
ssh largo
```

Expected `ssh -G largo` highlights:

```text
user derp
hostname largo.local
requesttty true
remotecommand /opt/homebrew/bin/bash -l
```

## Related Notes
- [[Machine Profile - largo]]
- [[Machine Profile - adagio]]
- [[SSH Remote Access Recovery and Operations Manual (Windows/Linux/macOS)]]
