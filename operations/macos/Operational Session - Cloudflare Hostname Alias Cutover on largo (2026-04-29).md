---
title: Operational Session - Cloudflare Hostname Alias Cutover on largo (2026-04-29)
type: session
entity_type: session
permalink: agent-kb/operations/macos/operational-session-cloudflare-hostname-alias-cutover-on-largo-2026-04-29
created: 2026-04-29
modified: 2026-05-28
status: archive
tags:
- project/agent-kb
- domain/cloudflare
- domain/ssh
- domain/browser-rendering
- domain/macos
- machine/largo
- status/archive
- date/2026-04-29
---

# Operational Session - Cloudflare Hostname Alias Cutover on largo (2026-04-29)

## Objective
Add machine-scoped hostnames for the existing browser-rendered terminal stack:
- `ssh.largo.braisenly.com`
- `opencode.largo.braisenly.com`

without removing existing hostnames.

## Control-plane actions completed
1. Updated tunnel `3227fecc-8f59-41e4-9501-ab8235bc42f3` ingress rules to add:
   - `ssh.largo.braisenly.com -> ssh://localhost:22`
   - `opencode.largo.braisenly.com -> http://127.0.0.1:4096`
2. Kept existing ingress rules intact:
   - `ssh.braisenly.com`
   - `opencode.braisenly.com`
3. Added proxied DNS CNAME records:
   - `ssh.largo.braisenly.com -> 3227fecc-8f59-41e4-9501-ab8235bc42f3.cfargotunnel.com`
   - `opencode.largo.braisenly.com -> 3227fecc-8f59-41e4-9501-ab8235bc42f3.cfargotunnel.com`

## Observed results
- Tunnel API update succeeded and tunnel config source became `cloudflare`.
- DNS creation succeeded for both hostnames.
- Existing root/local health still showed `localhost:22` open.
- `localhost:4096` remained closed on this Mac during the session.
- HTTPS probes to both new nested hostnames failed with TLS handshake alerts.

## Key inference
The nested hostnames likely failed because certificate coverage did not include `*.largo.braisenly.com` at the time of validation. Standard `*.braisenly.com` coverage does not automatically include nested wildcard labels.

## Follow-up actions required
1. Provision certificate coverage for `*.largo.braisenly.com` (Cloudflare advanced cert path), then re-test both hostnames.
2. If rapid recovery is needed before cert changes, use one-label hostnames (for example `ssh-largo.braisenly.com`, `opencode-largo.braisenly.com`) mapped to the same tunnel.
3. For `opencode` functional availability, ensure an origin is actually listening on `127.0.0.1:4096` on the intended host.

## 2026-04-30 addendum
- One-label fallback hostnames were fully cut in and validated:
  - `ssh-largo.braisenly.com` -> `302` (Access redirect)
  - `opencode-largo.braisenly.com` -> `302` (Access redirect)
- Access apps were created for the one-label fallback hostnames:
  - `largo-ssh-dash`
  - `largo-opencode-dash`
- Nested aliases still fail at TLS handshake before Access:
  - `ssh.largo.braisenly.com`
  - `opencode.largo.braisenly.com`
- Attempted advanced certificate ordering via API from available local tokens failed due auth/scope mismatch; an SSL-write token is still required.

## Evidence summary
- Tunnel config API read/write via account endpoint.
- DNS record create/read via zone endpoint.
- Local checks:
  - `localhost:22` open
  - `localhost:4096` closed
- Edge checks:
  - `curl -I https://ssh.largo.braisenly.com` -> TLS handshake failure
  - `curl -I https://opencode.largo.braisenly.com` -> TLS handshake failure

## Relations
- related_to [[Cloudflare Browser Rendering on largo]]
- related_to [[Runbook - Cloudflare Browser Rendering Terminal Verification and Recovery on largo]]
- related_to [[Machine Profile - largo]]
- related_to [[Index - Cloudflare Browser Rendering Operations]]
