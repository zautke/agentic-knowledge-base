---
title: Injectable Provider Tool Loop Pattern – wxt-prompt
type: pattern
permalink: agent-kb/projects/wxt-prompt/patterns/injectable-provider-tool-loop-pattern-wxt-prompt
created: 2026-04-21
modified: 2026-04-21
entity_type: pattern
status: evergreen
tags:
- project/wxt-prompt
- domain/browser-automation
- pattern/testability
- status/evergreen
---

# Injectable Provider Tool Loop Pattern – wxt-prompt

## Context

The wxt-prompt extension implements agentic tool calling for Ollama, Anthropic, and OpenAI. Each provider has a `toolLoop.ts` file with a pure async function that drives the tool-use loop.

## The Pattern

Each tool loop takes `chat` and `toolExecutor` as injected dependencies — not imported directly:

```typescript
// src/providers/anthropic/toolLoop.ts
export async function runAnthropicToolLoop(args: {
  model: string;
  messages: AnthropicMessage[];
  tools: AnthropicTool[];
  chat: (req: { model: string; messages: AnthropicMessage[]; tools: AnthropicTool[] }) => Promise<AnthropicResponse>;
  toolExecutor: (name: string, toolArgs: Record<string, unknown>) => Promise<unknown>;
}): Promise<{ textContent: string; messages: AnthropicMessage[] }>
```

The `client.ts` provides the real `chat` closure (a `fetch` call with `stream: false`).  
The `useAgentLoop.ts` provides the real `toolExecutor` (`browser.runtime.sendMessage → EXECUTE_TOOL`).

## Why this matters

The loop logic is **fully unit-testable in Vitest with no browser**:

```typescript
it('executes tool call and loops', async () => {
  const mockChat = vi.fn()
    .mockResolvedValueOnce({ stop_reason: 'tool_use', content: [{ type: 'tool_use', id: 'tu_1', name: 'get_tab_info', input: {} }] })
    .mockResolvedValueOnce({ stop_reason: 'end_turn', content: [{ type: 'text', text: 'Tab is example.com' }] });

  const toolExecutor = vi.fn().mockResolvedValue({ url: 'https://example.com' });

  const result = await runAnthropicToolLoop({ model: '...', messages, tools, chat: mockChat, toolExecutor });
  expect(result.textContent).toBe('Tab is example.com');
  expect(mockChat).toHaveBeenCalledTimes(2);
});
```

The only untestable part in unit tests is the `browser.runtime.sendMessage` bridge in `toolExecutor`.

## File locations

| File | Role |
|------|------|
| `src/providers/anthropic/toolLoop.ts` | Anthropic loop — exported `AnthropicTool`, `AnthropicMessage`, `AnthropicResponse` |
| `src/providers/openai/toolLoop.ts` | OpenAI loop — exported `OpenAITool`, `OpenAIMessage`, `OpenAIResponse` |
| `src/providers/ollama/toolLoop.ts` | Ollama loop — original reference pattern |
| `src/core/tools/browserToolSchemas.ts` | Single source: `BROWSER_TOOL_SCHEMAS`, `toAnthropicTools()`, `toOpenAITools()` |
| `src/core/agent-loop/useAgentLoop.ts` | Gate: `TOOL_CAPABLE_PROVIDERS = new Set(['ollama','anthropic','openai'])` |

## Hardening checklist (learned from curmudgeon reviews)

- `toolExecutor` must be wrapped in try/catch — push error as tool result, loop continues
- Loop exhaustion must `throw`, not `return { textContent: '' }` (silent failure indistinguishable from empty response)
- Anthropic: `is_error: true` in tool_result block on executor failure
- OpenAI: `content: 'Error: <message>'` in `role: 'tool'` message on executor failure
- Export types (`AnthropicMessage`, `AnthropicResponse`) so `client.ts` can use them without `Parameters<typeof ...>` casts
- `stop_reason` / `finish_reason` should be narrowed to literal union — prevents silent `max_tokens` terminal masquerade

## Relations

- part_of [[wxt-prompt – Project Overview]]
- extends [[Browser Extension CDP Testing Patterns]]
- implements [[Agentic Browser Automation Overview]]

## Live Browser Verification (2026-04-21)

**Endpoint:** `https://ollama.braisenly.com` | **Model:** `deepseek-v3.1:671b-clo`

All scenarios passed:
- DevTools toggle persists via Dexie across sidepanel reload ✅
- System prompt injection confirmed: response correctly named `example.com` ✅
- Tool call confirmed: `get_tab_info` returned "Example Domain" page title ✅
- Provider probe resolved in **5 seconds** (remote endpoint) ✅

**Timing note for future agents**: Provider connection time varies dramatically by endpoint. Remote Ollama resolved in 5s. Local `localhost:11434` may take 60+ seconds — always poll for up to 180s before concluding provider is unavailable.

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|---------|---------|
| 2026-04-21 | Created | Active tab context milestone complete — document injectable loop pattern and hardening checklist | Post-implementation curation |
| 2026-04-21 | Added live verification results | Browser test confirmed system prompt + tool call working end-to-end | Browser tester agent run |
