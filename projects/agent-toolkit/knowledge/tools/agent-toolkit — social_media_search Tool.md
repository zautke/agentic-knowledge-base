---
title: agent-toolkit — social_media_search Tool
type: reference
permalink: agent-kb/projects/agent-toolkit/knowledge/tools/agent-toolkit-social-media-search-tool
created: 2026-04-18
modified: 2026-04-18
entity_type: reference
status: evergreen
tags:
- project/agent-toolkit
- domain/agent-engineering
- domain/web-search
- domain/social-media
- status/evergreen
---

# social_media_search Tool

## What It Does
Searches social media platforms using Tavily's domain filtering. Optionally fetches full content from result URLs via a secondary extract call. Synchronous (not async).

**Source**: `tools/social_media.py::social_media_search` — `local/agent-toolkit-49aa8c6c`

## Platform Support
```python
PLATFORM_DOMAINS = {
    "reddit":    "reddit.com",
    "x":         "twitter.com",
    "linkedin":  "linkedin.com",
    "tiktok":    "tiktok.com",
    "instagram": "instagram.com",
    "facebook":  "facebook.com",
}
# "combined" → all domains in include_domains
```

## Signature
```python
def social_media_search(
    query: str,
    api_key: str,
    platform: Literal["tiktok","facebook","instagram","reddit","linkedin","x","combined"] = "combined",
    include_raw_content: bool = False,   # if True: secondary extract call
    max_results: int = 5,
    include_answer: bool = False,
    include_images: bool = False,
    time_range: Optional[Literal["day","week","month","year"]] = None,
    max_retries: int = 1,
) -> Dict[str, Any]
```

## Two-Phase Pattern (when `include_raw_content=True`)
```
Phase 1: Tavily search (always basic depth, domain-filtered)
   → Returns titles, URLs, snippets
Phase 2: extract_with_retry(urls, extract_depth="advanced")
   → Populates result["raw_content"] and result["images"]
   → If extraction fails: result["extraction_error"] = str(e), raw_content=None
```

**Note**: Phase 1 always sets `include_raw_content=False` — raw content is fetched manually in Phase 2 to allow `extract_depth="advanced"` (tables, embedded content).

## Implementation Note: Synchronous
Unlike other tools which are `async def`, `social_media_search` is synchronous (`def`). Caller must not `await` it. This reflects that the underlying `search_with_retry` is synchronous.

## Usage Tracking
- Phase 1: `usage.tavily.add_search(credits, response_time)`
- Phase 2 (if raw content): `usage.tavily.add_extract(credits, response_time)`
- Full `usage` dict appended to response

## Relations
- part_of [[agent-toolkit — README]]
- uses [[agent-toolkit — Observability Data Model]]