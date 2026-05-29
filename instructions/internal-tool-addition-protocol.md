---
title: internal-tool-addition-protocol
type: instruction
permalink: agent-kb/instructions/internal-tool-addition-protocol
created: 2026-03-09
modified: 2026-03-09
entity_type: instruction
status: evergreen
tags:
- project/agent-kb
- domain/tooling
- domain/agent-protocol
- status/evergreen
- source/largo
- machine/largo
---

# Internal Tool Addition Protocol

Deterministic procedure for adding internal tools so provider behavior, schema exposure, and verification remain aligned.

## Governing Rules
- Tool name must be stable snake_case and action-oriented.
- Runtime validation and model-facing JSON schema must match exactly.
- Tool outputs must be structured JSON-safe payloads.
- Provider wiring, Ollama exposure, and tests are mandatory.
- Completion requires verification evidence (`vitest`, `typecheck`, `build`).

## Deterministic Steps
1. Define provider contract and typed request/response payload.
2. Implement `ensure*Params` guard with defaults + fast-fail validation.
3. Implement core behavior in helper functions; keep `execute` thin.
4. Register tool in Ollama exposure with explicit schema and enums.
5. Verify execution path resolves by exact tool name.
6. Add tests:
- provider success + validation failure
- exposure schema + enum/property assertions
- format/output-specific assertions for encoded/binary outputs
7. Run gates:
- targeted `vitest`
- `pnpm typecheck`
- `pnpm build:chrome`
8. Record protocol evolution and edge-cases in docs/KB.

## Canonical Implementation Prompt
"Add a new internal tool using the Internal Tool Addition Protocol.

Requirements:
1. Implement tool contract and runtime validation in provider code.
2. Implement deterministic helper functions with typed request/response.
3. Expose matching JSON schema in `src/providers/ollama/toolExposure.ts`.
4. Add provider and exposure tests (success + failure + schema assertions).
5. Run verification gates: targeted vitest, `pnpm typecheck`, `pnpm build:chrome`.
6. Return changed files, verification evidence, and unresolved risk.

Hard constraints:
- Keep changes minimal and localized.
- No `any` types.
- No schema/runtime mismatch.
- No completion claims without verification evidence."

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-03-09 | Initial protocol added for deterministic internal tool integration. | Needed consistent repeatable process for adding tools like `as_downloadable` with full provider+schema+test parity. | User request for deterministic tool-addition prompt. |

## Relations

- implements [[Action Taxonomy – Index]]
- related_to [[Document Curation Metaprompt]]
- related_to [[Agentic Knowledge Base System Overview]]