---
title: Agentic Teams Repository
created: 2026-02-16
modified: 2026-02-16
type: index
entity_type: index
status: evergreen
permalink: teams/agentic-teams-repository
tags:
- project/agent-kb
- domain/teams
- domain/agent-coordination
- index/registry
- status/evergreen
---

# Agentic Teams Repository

Central registry of all agent teams available for complex multi-agent missions.

---

## Active Teams

| ID | Name | Domain | Agents | Status |
|----|------|--------|--------|--------|
| DS-001 | Design System Squad | Design Systems, Theming, Components | 10 | ✅ Active |
| UX-001 | UI/UX Overhaul Strike Force | Frontend Design, UX Engineering, Visual Excellence | 7 | ✅ Active |
| TEST-001 | Quality Assurance Squad | Testing, Validation, Code Quality | 6 | ✅ Active |

---

## Team Categories

### Development Teams
- [[DS-001 Design System Squad]] - Design systems, component libraries, theming
- [[UX-001 UI Overhaul Strike Force]] - Elite UI/UX transformation, award-winning aesthetics

### Quality Assurance Teams
- [[TEST-001 Quality Assurance Squad]] - E2E testing, type safety, linting, coverage analysis

### Infrastructure Teams
(None registered yet)

### Research Teams
(None registered yet)

---

## Quick Commands

```bash
# List all teams
/list-teams

# Spawn a team for a mission
/spawn-team DS-001 "Add Ocean theme palette"

# Create a new team
/team-guide

# View team details
/list-teams DS-001 --verbose
```

---

## Team Creation Guidelines

See [[TeamSpawning]] for creating new teams.

Required elements:
1. Unique ID (format: XXX-NNN)
2. 4-10 agents with defined roles
3. Execution phases (parallel/sequential)
4. Quality gates and protocols
5. Spawning examples

---

## Statistics

- **Total Teams**: 3
- **Total Agents**: 23
- **Missions Completed**: 0
- **Last Updated**: 2026-02-16

---

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-02-16 | Metadata and link hygiene pass | Normalized frontmatter to the KB metadata contract and replaced unresolved links with canonical or registry-safe references. | KB hygiene sweep triggered by recent activity. |
| 2026-02-16 | DS-001 canonical link restoration | Re-linked DS-001 entries after re-establishing the missing team node so index navigation remains complete. | Follow-up pass after protocol refile and DS-001 node creation. |
| 2026-02-16 | Registry relation completion | Added explicit `contains` relation for TEST-001 so relation block fully matches active-team index entries. | Follow-up consistency check during hygiene sweep. |

---

## Relations
- contains [[DS-001 Design System Squad]]
- contains [[UX-001 UI Overhaul Strike Force]]
- contains [[TEST-001 Quality Assurance Squad]]
- defines [[TeamSpawning]]