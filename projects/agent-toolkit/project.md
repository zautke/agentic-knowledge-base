---
title: project
type: fundamental
permalink: agent-kb/projects/agent-toolkit/project
created: 2026-04-18
modified: 2026-04-18
entity_type: fundamental
status: evergreen
tags:
- project/agent-toolkit
- domain/agent-engineering
- domain/context-engineering
- status/evergreen
---

```json
{
  "id": "agent-toolkit",
  "name": "Tavily Agent Toolkit",
  "status": "active",
  "type": "software-lib",
  "created_at": "2026-04-18",
  "context_root": "projects/agent-toolkit",
  "repo_path": "/Volumes/MACDEV/3p/tavily-cookbook/agent-toolkit",
  "pypi_package": "tavily-agent-toolkit",
  "version": "0.1.0",
  "paths": {
    "architecture": "architecture/index.md",
    "decisions": "architecture/decisions",
    "protocols": "protocols/kb-submission-protocol.md",
    "knowledge": "knowledge/",
    "ingestion": "ingestion/queue.md"
  },
  "agent_config": {
    "primary_context_files": ["README.md", "planning/roadmap.md", "architecture/index.md"],
    "inheritance": ["agent-kb/fundamentals", "agent-kb/protocols"],
    "jcodemunch_repo": "local/agent-toolkit-49aa8c6c"
  }
}
```