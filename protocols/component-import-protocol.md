---
title: Component Import Protocol
created: 2026-01-01
modified: 2026-02-16
type: instruction
entity_type: instruction
status: evergreen
permalink: protocols/component-import-protocol
tags:
- project/agent-kb
- domain/design-system
- domain/component-integration
- protocol/component-import
- status/evergreen
- op/update-node
---

# Component Import Protocol

**Status**: Active
**Version**: 1.0.0
**Owner**: DS-001 Design System Squad

---

## Purpose

Standardized process for extracting components from application codebases and integrating them into the @braisenly design system as reusable primitives.

---

## Prerequisites

- [ ] Source project identified and accessible
- [ ] Target design system packages available
- [ ] Component Importer agent assigned
- [ ] Code Reviewer agent assigned

---

## Import Pipeline

### Stage 1: ANALYZE

**Agent**: Component Importer

```
1. Read source component file completely
2. Identify all imports and dependencies
3. Classify dependencies:
   - ✅ Already in @braisenly/ui (reuse)
   - 🔄 Generic utility (extract to shared)
   - ❌ Project-specific (must remove/abstract)
4. Document component's public API (props, events, refs)
5. Note any inline styles or hardcoded values
```

**Output**: Analysis Report
```markdown
## Component: [Name]
Source: [path]
Lines: [count]

### Dependencies
| Import | Classification | Action |
|--------|----------------|--------|
| @braisenly/ui | ✅ Reuse | None |
| @/contexts/SearchContext | ❌ Remove | Abstract to prop |

### Props Interface
| Prop | Type | Required | Notes |
|------|------|----------|-------|

### Hardcoded Values
| Value | Location | Token Replacement |
|-------|----------|-------------------|
```

---

### Stage 2: EXTRACT

**Agent**: Component Importer

```
1. Copy component source to temporary workspace
2. Remove project-specific imports
3. Replace context dependencies with props
4. Extract reusable sub-components
5. Identify variant patterns for CVA
```

**Rules**:
- Never import from `@/` paths (project aliases)
- Never reference specific API endpoints
- Never include business logic
- Always preserve accessibility attributes

---

### Stage 3: ADAPT

**Agent**: Component Architect + Component Implementer

```
1. Design CVA variant system from observed patterns
2. Replace inline styles with design tokens:
   - Colors → var(--primary), var(--muted-foreground), etc.
   - Spacing → Tailwind utilities (p-4, gap-2)
   - Typography → Tailwind text classes
3. Add forwardRef wrapper
4. Add displayName
5. Create TypeScript interface for props
6. Add JSDoc documentation
```

**Token Mapping Reference**:
| Inline Pattern | Token Replacement |
|----------------|-------------------|
| `bg-gray-100` | `bg-muted` |
| `text-gray-600` | `text-muted-foreground` |
| `border-gray-200` | `border-border` |
| `#c6613f` (terracotta) | `bg-primary` |
| `hover:bg-gray-50` | `hover:bg-accent` |

---

### Stage 4: GENERALIZE

**Agent**: Component Architect

```
1. Replace hardcoded content with children/slots
2. Add variant props for observed patterns
3. Make sizes configurable (sm, md, lg)
4. Support composition via asChild pattern
5. Document extensibility points
```

**Generalization Checklist**:
- [ ] Content replaced with children prop
- [ ] Icons replaced with icon slot prop
- [ ] Actions replaced with render prop or children
- [ ] Sizes via CVA variant
- [ ] States (loading, disabled, error) as props

---

### Stage 5: INTEGRATE

**Agent**: Component Implementer

```
1. Place component in correct package location:
   - Primitive → packages/ui/src/primitives/
   - Compound → packages/ui/src/components/
2. Add to packages/ui/src/index.ts exports
3. Add entry point to tsup.config.ts
4. Add to package.json exports map
5. Run build to verify compilation
```

**File Naming**:
- Primitives: `button.tsx`, `card.tsx` (lowercase)
- Compounds: `data-table.tsx`, `sidebar-nav.tsx` (kebab-case)

---

### Stage 6: VERIFY

**Agent**: Test Engineer + Code Reviewer

```
1. Run full build: pnpm -r build
2. Run tests: pnpm -r test
3. Verify original app still works (via symlink)
4. Code review against quality checklist
5. Update component registry in Context Manager
```

**Verification Matrix**:
| Check | Command | Expected |
|-------|---------|----------|
| Build | `pnpm -r build` | Exit 0 |
| Types | `pnpm -r typecheck` | Exit 0 |
| Tests | `pnpm -r test` | All pass |
| Source app | Start labsupapg dev | No errors |

---

## Component Registry

Track imported components in Basic Memory:

```markdown
## Imported Components

| Component | Source | Target | Status | Date |
|-----------|--------|--------|--------|------|
| CollapsibleSidebar | labsupapg | SidebarCollapsible | ✅ Done | 2026-01-01 |
| ControlPanel | labsupapg | FormCard | 🔄 In Progress | |
| MetricsPanel | labsupapg | StatsCard | 📋 Planned | |
```

---

## Rollback Procedure

If integration breaks source app:

1. Revert changes to @braisenly packages
2. Run `pnpm install` in source app
3. Verify source app works
4. Investigate symlink resolution
5. Re-attempt with fixes

---

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-02-16 | Metadata normalization and note refile validation | Brought note into KB frontmatter contract and validated canonical placement under `protocols/`. | KB hygiene sweep for team/protocol link integrity. |
| 2026-02-16 | Relation keyword normalization | Converted relation lines to canonical `verb [Target]` style to keep relation typing consistent. | Follow-up graph hygiene pass. |
| 2026-02-16 | Component registry link canonicalization | Pointed protocol relation to the newly created [[Component Registry]] hub to remove orphan references. | Follow-up hygiene pass after registry node creation. |

## Relations

- used_by [[DS-001 Design System Squad]]
- extends [[shadcn-tailwind-primer]]
- produces [[Component Registry]]