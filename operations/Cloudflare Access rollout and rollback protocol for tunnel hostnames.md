---
title: Cloudflare Access rollout and rollback protocol for tunnel hostnames
type: note
permalink: kb/operations/cloudflare-access-rollout-and-rollback-protocol-for-tunnel-hostnames
tags:
- operation
- cloudflare
- access
- tunnel
- rollback
- runbook
---

# Cloudflare Access rollout and rollback protocol for tunnel hostnames

## Purpose
Provide a future-agent-safe procedure for placing Cloudflare Access in front of public tunnel hostnames and, critically, for reverting it without cutting off access to operational systems such as `kb`.

This note exists because an attempted rollout protected three hosts successfully, but it also blocked other agents from reaching `kb` until the Access apps were removed. The rollback path must therefore be treated as first-class, not as an afterthought.

## Hosts involved in the April 19 2026 rollout
- `ollama.braisenly.com`
- `opencode.braisenly.com`
- `abmex-kb.braisenly.com`

## What was created during the rollout
### Access apps
- `ollama-api`
  - app id: `b9f9c12a-671d-4b31-9a3f-e20b2974ef85`
- `opencode`
  - app id: `2603c11a-f551-41e4-9156-f07dc9f2fcb0`
- `abmex-kb`
  - app id: `cb0d6d38-497b-4d06-8124-df5d986c8ad6`

### Access policies
Each app had a policy with:
- `decision: non_identity`
- `include: [{ service_token: { token_id: ... } }]`

### Service token
- token id: `8e5bd4ed-86a3-4d00-b9f3-9684cbf92830`
- name: `braisenly-tunnel-service`

## What worked
The Access rollout itself worked correctly.

Verification results during rollout:
- `ollama`: unauthenticated request returned `403`, authenticated service-token request returned `200` on `/v1/models`
- `opencode`: unauthenticated request returned `403`, authenticated service-token request returned `200` on `/`
- `abmex-kb`: unauthenticated request returned `403`; authenticated request reached origin and returned `404`, proving Access was enforced and then bypassed correctly

## Why rollback was required
Even though the Cloudflare Access configuration behaved correctly, protecting `abmex-kb` caused other agents to lose access to the KB system. That made the protection operationally incorrect for this environment.

The lesson is:
- do not put Access in front of `kb` or other shared agent infrastructure unless every agent path is already updated to send valid Cloudflare Access service-token headers
- if uncertain, protect consumer endpoints first and defer shared infrastructure

## Required permissions for a future rollout
At minimum, the automation token needs:
- `Account -> Access: Apps and Policies -> Edit`
- `Account -> Access: Service Tokens -> Edit`

Helpful additional permissions:
- `Account -> Access: Apps and Policies -> Read`
- `Account -> Access: Service Tokens -> Read`

For tunnel and hostname setup, separate tokens/permissions may also be needed:
- `Account -> Cloudflare Tunnel -> Edit`
- `Zone -> DNS -> Edit`

## Safe rollout procedure
### 1. Inventory target hosts and classify them
Before creating any Access objects, classify each hostname:
- shared infrastructure used by agents or automation
- human-facing application
- machine-to-machine API endpoint

Rule:
- if the hostname is shared infrastructure, assume breakage until proven otherwise

### 2. Verify the Access automation token
For account-owned `cfat_...` tokens, verify against the account endpoint, not the legacy user endpoint.

Pattern:
- `GET /accounts/{account_id}/tokens/verify`

Why:
- future agents can waste time diagnosing a valid token as invalid if they hit the wrong verification endpoint

### 3. Inspect existing Access apps before creating anything
Call:
- `GET /accounts/{account_id}/access/apps`

Why:
- avoid duplicate apps
- align with existing account conventions
- capture existing app ids for later cleanup if rollback is needed

### 4. Create one self-hosted Access app per hostname
Use one app per hostname. Do not bundle multiple hostnames into one app.

Reason:
- per-host rollback is straightforward
- blast radius is smaller
- audience boundaries stay clean

### 5. Create a dedicated service token for the rollout
Call:
- `POST /accounts/{account_id}/access/service_tokens`

Reason:
- one token can authenticate a controlled set of rollout tests
- it is easier to delete later than to reuse a broad preexisting secret

### 6. Attach a service-token policy to each app
Policy shape:
- `decision: non_identity`
- include the service token id

Reason:
- this is the Cloudflare API representation of service-auth access
- it is the correct way to protect machine-oriented services

### 7. Verify in two phases for every host
For each host, do both:
- unauthenticated request: confirm `403`
- authenticated request with service-token headers: confirm either `200` or some origin response that proves the request passed Access

Important:
- a post-auth `404` can still be a successful Access verification if the unauthenticated request was `403`
- the transition from edge block to origin response is the signal

### 8. Special rule for KB or shared agent infrastructure
Do not stop after host-local verification.

Also verify:
- at least one real external agent path can still reach the system
- at least one KB write or read path works from the tooling that depends on it

If this is not tested, the rollout is incomplete.

## Safe rollback procedure
Rollback must be executable in this order.

### 1. Unblock shared infrastructure first
If `kb` or another shared host is involved, remove its Access app before the others.

Reason:
- restore operational tooling before completing cleanup elsewhere

### 2. Delete the Access apps
Call:
- `DELETE /accounts/{account_id}/access/apps/{app_id}`

During the April 19 2026 rollback, the following app ids were deleted successfully:
- `b9f9c12a-671d-4b31-9a3f-e20b2974ef85`
- `2603c11a-f551-41e4-9156-f07dc9f2fcb0`
- `cb0d6d38-497b-4d06-8124-df5d986c8ad6`

### 3. Delete the service token after all apps are gone
Call:
- `DELETE /accounts/{account_id}/access/service_tokens/{token_id}`

During the April 19 2026 rollback, the deleted service token id was:
- `8e5bd4ed-86a3-4d00-b9f3-9684cbf92830`

Reason:
- keep secrets tidy
- ensure no stale rollout credential remains after the protection is removed

### 4. Verify the hosts are open again without headers
Expected post-rollback behavior observed on April 19 2026:
- `https://abmex-kb.braisenly.com/` -> `404` without Access headers
- `https://abmex-kb.braisenly.com/health` -> `404` without Access headers
- `https://ollama.braisenly.com/v1/models` -> `200` without Access headers
- `https://opencode.braisenly.com/` -> `200` without Access headers

Interpretation:
- no Cloudflare Access gate remains in front of these hosts
- responses are now direct origin behavior again

## Failure modes to recognize quickly
### Symptom: unauthenticated request returns `403` with `Cf-Access-*` headers
Meaning:
- Access is active on that hostname

### Symptom: authenticated request returns `200`
Meaning:
- Access accepted the service token and origin returned success

### Symptom: authenticated request returns `404`
Meaning:
- Access accepted the service token and origin did not recognize the route
- this is not an Access failure

### Symptom: KB tools start returning Cloudflare Access HTML instead of JSON
Meaning:
- KB is behind Access and the agent path is not sending valid service-token credentials
- rollback should be considered immediately unless the entire toolchain is already Access-aware

## Recommendations for future agents
- never protect `kb` first
- test `kb` read/write flows explicitly before declaring success
- record app ids and service-token id at creation time so rollback is deterministic
- prefer narrow rollout scope over broad rollout scope
- treat rollback as part of the initial implementation plan, not as emergency improvisation

## April 19 2026 result
The Access rollout was fully reverted.

After rollback:
- `abmex-kb` was unblocked first
- `ollama` and `opencode` were also restored to open access
- the service token was deleted
- shared-agent access to KB was restored
