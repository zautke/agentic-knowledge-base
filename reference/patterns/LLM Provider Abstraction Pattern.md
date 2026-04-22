---
title: LLM Provider Abstraction Pattern
type: pattern
permalink: agent-kb/reference/patterns/llm-provider-abstraction-pattern
created: 2026-04-18
modified: 2026-04-18
entity_type: pattern
status: evergreen
tags:
- domain/agent-engineering
- domain/llm-reliability
- domain/patterns
- status/evergreen
---

# LLM Provider Abstraction Pattern

## Summary
A configuration layer that decouples agent/tool code from any specific LLM provider. Uses LangChain's `init_chat_model` to support 20+ providers through a single `ModelConfig`/`ModelObject` interface. Provider is inferred from model name prefix or specified explicitly.

## Problem
Agents hardcoded to a single provider are fragile — API changes, deprecations, cost spikes, and compliance requirements force rewrites. Tests become coupled to provider availability.

## Solution: ModelObject + ModelConfig

### ModelObject — Provider-Agnostic Model Config
```python
@dataclass
class ModelObject:
    model: str                           # "gpt-5" or "openai:gpt-5" or "claude-*"
    model_provider: Optional[str] = None # explicit override
    max_tokens: Optional[int] = None
    api_key: Optional[str] = None        # per-model key (multi-tenant safe)
```

**Provider inference from model name prefix** (via LangChain):
| Prefix | Inferred Provider |
|---|---|
| `gpt-*`, `o1*`, `o3*` | openai |
| `claude*` | anthropic |
| `gemini*` | google_vertexai |
| `command*` | cohere |
| `mistral*` | mistralai |
| `deepseek*` | deepseek |
| `grok*` | xai |
| `sonar*` | perplexity |
| `amazon.*` | bedrock |
| `accounts/fireworks*` | fireworks |

### ModelConfig — Primary + Fallback Chain
```python
@dataclass
class ModelConfig:
    model: ModelObject
    fallback_models: Optional[list[ModelObject]] = None
    temperature: Optional[float] = None
    timeout: Optional[float] = None

    def get_all_models(self) -> list[ModelObject]:
        return [self.model] + (self.fallback_models or [])

    def to_init_kwargs(self, model_object=None) -> dict:
        # Builds kwargs for init_chat_model, excluding None values
        # Handles model, model_provider, temperature, max_tokens, timeout, api_key
```

## Usage Pattern
```python
# Simple
config = ModelConfig(model=ModelObject(model="gpt-5"))

# With fallback cascade
config = ModelConfig(
    model=ModelObject(model="gpt-5", api_key="..."),
    fallback_models=[
        ModelObject(model="claude-sonnet-4-5-20250514"),
        ModelObject(model="gemini-2.5-flash"),
    ],
    temperature=0.7
)

# Pass to any tool or agent
result = await search_and_answer(api_key=..., query=..., model_config=config)
```

## Key Properties
- **Per-model API keys** — enables multi-tenant and multi-account configurations
- **Provider format**: `"provider:model"` string or inferred from prefix
- **`None` exclusion** in `to_init_kwargs` — only set fields are passed to LangChain
- **Integrates with [[LLM Fallback Cascade Pattern]]** via `get_all_models()`

## Known in These Repos
- [2026-04-18] `tavily-agent-toolkit` v0.1.0 — canonical implementation — `models.py:289-371`

## Evolution Log

| Date | Change | Reason | Trigger |
|---|---|---|---|
| 2026-04-18 | Initial pattern documented | Extracted from `ModelConfig`/`ModelObject` during agent-toolkit KB seeding | Global truth promotion — Bridge Protocol |

## Relations
- derived_from [[agent-toolkit — README]]
- related_to [[LLM Fallback Cascade Pattern]]
- used_by [[Hybrid Intelligence Pattern]]