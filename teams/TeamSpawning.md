---
title: TeamSpawning
created: 2026-01-01
modified: 2026-02-16
type: instruction
entity_type: instruction
status: evergreen
permalink: teams/team-spawning
tags:
- project/agent-kb
- domain/teams
- domain/agent-coordination
- guide
- canonical
- op/update-node
---

# TeamSpawning: Agent Team Creation Guide

The canonical reference for creating multi-agent teams.

---

## Three-Tier Hierarchy

Every team follows this structure:

```
┌─────────────────────────────────────────┐
│           🎯 ORCHESTRATOR               │
│         (Claude Code Primary)           │
│   - Mission decomposition               │
│   - Agent coordination                  │
│   - Decision authority                  │
└────────────────┬────────────────────────┘
                 │
    ┌────────────┴────────────┐
    ▼                         ▼
┌─────────────┐       ┌─────────────┐
│ 📋 CONTEXT  │       │ ✍️ SCRIBE   │
│   MANAGER   │       │             │
│ State sync  │       │ Docs/notes  │
│ Knowledge   │       │ Session log │
└─────────────┘       └─────────────┘
                 │
    ┌────────────┴────────────┐
    ▼                         ▼
┌─────────────────────────────────────────┐
│          🔧 SPECIALISTS (3-7)           │
│   Domain experts for specific tasks     │
└─────────────────────────────────────────┘
```

---

## Mandatory Roles

### 🎯 Orchestrator (Claude Code)
**Always the primary Claude instance**
- Receives mission from user
- Decomposes into agent-appropriate tasks
- Manages handoffs and dependencies
- Synthesizes final deliverables
- Makes tie-breaking decisions

### 📋 Context Manager
**subagent_type**: `context-manager`
- Maintains shared state across agents
- Persists decisions to Basic Memory
- Tracks progress and completion status
- Surfaces relevant prior knowledge

### ✍️ Scribe
**subagent_type**: `scribe` or `general-purpose`
- Documents session activities
- Creates session log in Basic Memory
- Captures decisions and rationale
- Generates handoff summaries

---

## Naming Formula

```
[Outcome] + [Approach] + [Descriptor]
```

**Components:**
| Part | Purpose | Examples |
|------|---------|----------|
| Outcome | What they achieve | Relevance, Performance, Security |
| Approach | How they work | Engineering, Analysis, Optimization |
| Descriptor | Team size/style | Squad (4-6), Unit (6-8), Force (8-12) |

**Codename Format**: `XXX-NNN`
- XXX = 2-3 letter domain code
- NNN = sequential number

**Examples:**
- `RES-001` Relevance Engineering Squad
- `DS-001` Design System Squad  
- `SEC-001` Security Audit Unit
- `PER-001` Performance Optimization Force

---

## Role Prompt Template

```markdown
### [Icon] [Role Name]
**subagent_type**: `[agent-type]`
**Role**: [One-line description]
**Prompt**:
\`\`\`
You are the [Role Name] for [Team Name].

Responsibilities:
1. [Primary responsibility]
2. [Secondary responsibility]
3. [Tertiary responsibility]

Key constraints:
- [Constraint 1]
- [Constraint 2]

Output format: [Expected deliverable format]
\`\`\`
```

---

## Specialist Role Icons

| Domain | Icon | Typical subagent_type |
|--------|------|----------------------|
| Orchestrator | 🎯 | (Claude primary) |
| Context | 📋 | context-manager |
| Scribe | ✍️ | scribe, general-purpose |
| Research | 🔬 | Explore |
| Architecture | 🏗️ | Plan, feature-dev:code-architect |
| Implementation | 💻 | frontend-developer, backend-developer, fullstack-developer |
| Code Review | 👁️ | feature-dev:code-reviewer |
| Testing | 🧪 | general-purpose |
| Security | 🔒 | general-purpose |
| Performance | ⚡ | general-purpose |
| Documentation | 📖 | scribe |
| DevOps/Release | 🚀 | backend-developer |
| Design/Theme | 🎨 | frontend-developer |

---

## Spawning Patterns

### Parallel Spawning
Use when tasks are independent:
```
Phase 1 (Parallel):
├─ Research Agent → Web research
├─ Code Agent → Codebase analysis  
└─ Context Agent → Load prior knowledge
```

### Sequential Spawning
Use when tasks depend on prior outputs:
```
Phase 2 (Sequential):
Architect → designs API
    ↓
Implementer → builds from design
    ↓
Reviewer → validates implementation
```

### Hybrid Pattern
Most teams use both:
```
Phase 1: Parallel research/analysis
Phase 2: Sequential design → implement
Phase 3: Parallel testing/documentation
Phase 4: Sequential review → release
```

---

## Team Document Template

```markdown
# [CODENAME]: [Team Name]

**Status**: ✅ Active | 🔧 Draft | 📦 Archived
**Version**: 1.0.0
**Created**: YYYY-MM-DD

---

## Team Identity

**Codename**: [XXX-NNN]
**Name**: [Full Team Name]
**Domain**: [Problem domain]
**Motto**: "[Inspirational phrase]"

---

## Mission Scope

[2-3 sentences describing what this team handles]

---

## Agent Roster

### 🎯 Orchestrator
[Standard orchestrator description]

### 📋 Context Manager
[Role prompt]

### ✍️ Scribe
[Role prompt]

### [Icon] [Specialist 1]
**subagent_type**: `[type]`
[Full role prompt]

[... additional specialists ...]

---

## Execution Phases

### Phase 1: [Name] (Parallel|Sequential)
| Agent | Task |
|-------|------|
| ... | ... |

[... additional phases ...]

---

## Protocols

[Links to relevant protocols]

---

## Spawning Command

\`\`\`
/spawn-team [CODENAME] "[mission]"
\`\`\`

---

## Relations
- implements [[relevant-document]]
- uses [[protocol]]
```

---

## Registration Checklist

Before registering a team:
- [ ] Has unique codename (XXX-NNN format)
- [ ] Includes Orchestrator, Context-Manager, Scribe
- [ ] 3-7 specialist roles defined
- [ ] Each role has complete prompt
- [ ] Spawning phases documented
- [ ] Saved to `teams/registered/[CODENAME]-[kebab-name].md`
- [ ] Added to Agentic Teams Repository index

---

## Related Commands

| Command | Purpose |
|---------|---------|
| `/spawn-team <id> "<mission>"` | Invoke a registered team |
| `/list-teams` | View all registered teams |
| `/list-teams <id> --verbose` | Detailed team view |
| `/adopt-skill` | Learn new domain before team creation |

---

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-02-16 | Metadata and relation hygiene pass | Normalized frontmatter to KB contract and replaced unresolved relation target with explicit protocol naming text. | Ongoing KB hygiene sweep for team documents. |

## Relations

- used_by [[Agentic Teams Repository]]