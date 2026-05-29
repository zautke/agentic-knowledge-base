---
modified: 2026-05-28
entity_type: context
title: UX-001 UI Overhaul Mission State
type: context
team: ux-001
created: 2026-01-20
status: initialized
permalink: agent-kb/teams/registered/ux-001/context/mission-state
tags:
- type/team-context
- team/ux-001
- ui-overhaul
- mission-state
- source/largo
- machine/largo
---

# UX-001 UI Overhaul Strike Force - Mission State

## Mission Objectives

### 1. "On This Page" Navigation Component
**Status**: Pending
**Priority**: High
- Create table-of-contents navigation for blog posts
- Extract TOC from contentlayer-generated headings
- Position: Sticky sidebar or floating panel
- Integration: PostLayout, PostSimple, PostBanner layouts

### 2. Jaw-Dropping Article Cards
**Status**: Pending
**Priority**: High
- Design visually stunning blog post preview cards
- Reference existing: `/Volumes/FLOUNDER/dev/blogg/components/ui/glass-card.tsx`
- Incorporate: featured images, gradient overlays, micro-interactions
- Integration: Home page, blog listing pages, tag pages

### 3. Theme System Replacement
**Status**: Pending
**Priority**: Critical
- Replace existing Material-UI v7 theme with new design system
- Consolidate: MUI theme + Tailwind v4 coexistence
- Current complexity: 50+ files in `/customTheme/` directory
- Preserve: CSS variables, color schemes (light/dark), SSR compatibility

### 4. Font Integration
**Status**: Partially Complete
**Priority**: Medium
**Fonts to Integrate**:
- Victor Mono NFM (monospace) - LOADED via next/font/google
- Rethink Sans (body text) - NEEDS INTEGRATION
- Rokkitt (display/headings) - LOADED via next/font/google

**Current Font Configuration**:
```typescript
// app/layout.tsx lines 6-45
const victorMono = Victor_Mono({ variable: '--font-victor-mono' })
const rokkitt = Rokkitt({ variable: '--font-rokkitt' })
const cormorant = Cormorant({ variable: '--font-cormorant' })
```

**Typography System Location**: `/Volumes/FLOUNDER/dev/blogg/customTheme/typography.ts`
- Uses Victor Mono as primary font
- Custom variants: banner (PalmBeach), poster (Cormorant), button (Barlow)

### 5. Quick-Publish Admin Page
**Status**: Pending
**Priority**: Low
- Create MDX creation interface
- Form fields: title, date, tags, summary, authors, layout
- Live preview of MDX content
- Save to `/data/blog/` directory
- Integration: Contentlayer automatic rebuild

---

## Prior Design Decisions from Basic Memory

### Design System Knowledge Base
Located: `/Users/luke/basic-memory/agent-kb/`

**Protocols Found**:
1. `/protocols/design-system-deployment-protocol.md`
   - Npm package publishing via GitHub Actions
   - pnpm workspace configuration
   - Version management with changesets
   - Status: DRAFT (last updated 2026-01-01)

2. `/operations/design-system-workflow.md`
   - CVA (class-variance-authority) for variants
   - Component checklist (forwardRef, displayName, tree-shaking)
   - Debugging patterns for tokens and tree-shaking
   - Status: DRAFT (last updated 2026-01-01)

**Key Insights**:
- Prior work on shadcn/Tailwind design system packaging
- Emphasizes tree-shakeable component architecture
- Uses @braisenly organization for npm packages
- NO SPECIFIC DECISIONS FOR THIS BLOG PROJECT YET

---

## Current Architecture Analysis

### Tech Stack Inventory
- **Framework**: Next.js 15.2.4 (App Router)
- **React**: 19.1.0
- **Styling**: Tailwind CSS v4 + Material-UI v7
- **Typography**: @tailwindcss/typography v0.5.16
- **Content**: Contentlayer v2 (0.5.4) with MDX
- **Animations**: Framer Motion v12.19.2, GSAP v3.13.0
- **Code Highlighting**: rehype-prism-plus, React Live v4.1.8
- **Icons**: Lucide React v0.525.0
- **Testing**: Playwright v1.55.0, Storybook v9.1.6

### Existing Theme System Complexity

**Color System** (`/customTheme/colorGuide.ts`):
- Dual palettes: `color` (light mode), `colorDark` (dark mode)
- Custom color tokens: treegreen, applegreen, deepgreen, walnut, burntorange, etc.
- MUI v7 integration via `colorSchemes` pattern

**Typography System** (`/customTheme/typography.ts`):
- Primary: Victor Mono Variable
- Custom variants: banner, poster, button
- Font fallback chain: Lato, system fonts
- Utility: `pxToRem()` for responsive sizing

**Component Overrides** (`/customTheme/componentOverrides/`):
- 22+ MUI component overrides (Avatar, Badge, Button, Card, etc.)
- Centralized in `/customTheme/index.ts`
- Uses deepmerge for composition

**Theme Export** (`/customTheme/index.ts`):
- `finalEnhancedTheme`: Main theme with CSS variables
- `createCustomTheme()`: Utility for custom overrides
- CSS variable prefix: `--mui-`
- Color scheme selector: `class` (uses .light/.dark classes)
- SSR compatible, performance optimized

**Tailwind Configuration** (`/tailwind.config.js`):
- Typography plugin extended with custom colors
- Font family configuration COMMENTED OUT (lines 8-12)
- Custom prose styles for light/dark modes

### Content Processing Pipeline

**Contentlayer Integration** (`/contentlayer.config.ts`):
- Blog schema: title, date, tags, summary, authors, layout, images
- Computed fields: readingTime, slug, path, toc (table of contents)
- Generates: `/app/tag-data.json`, `/public/search.json`
- TOC extraction: `extractTocHeadings()` from pliny/mdx-plugins

**MDX Component Mapping** (`/components/MDXComponents.tsx`):
```typescript
{
  Image, TOCInline, a: CustomLink,
  pre: Pre, table: TableWrapper,
  MDXCodeBlock, CodeBlock, CodeSample,
  GsapDemo, SimpleLoading,
  PlaygroundDemo, TestPlayground,
  ReloadableWrapper, SimpleCounter
}
```

**Interactive Code Features**:
- React Live integration for executable demos
- `ReloadContext` for component state management
- `PlaygroundActionPanel` with copy, reload, line numbers
- Code syntax highlighting via rehype-prism-plus

### Layout Templates
Located: `/Volumes/FLOUNDER/dev/blogg/layouts/`
- `PostLayout.tsx`: Full-featured with sidebar, tags, navigation
- `PostSimple.tsx`: Minimal design
- `PostBanner.tsx`: Hero image variant
- `ListLayout.tsx`: Blog index
- `ListLayoutWithTags.tsx`: Tag filtering
- `AuthorLayout.tsx`: Author pages

### Site Metadata (`/data/siteMetadata.js`):
- Theme: `system` (respects OS preference)
- Sticky nav: ENABLED
- Analytics: Umami
- Comments: Giscus
- Search: Kbar with `/search.json`
- Newsletter: Buttondown

---

## Component Registry Template

### Components to Create

#### 1. OnThisPageNav
**Purpose**: Table of contents navigation for blog posts
**Location**: `/components/new/OnThisPageNav.tsx`
**Props**:
```typescript
interface OnThisPageNavProps {
  headings: Array<{ url: string; depth: number; value: string }>
  currentSlug?: string
}
```
**Dependencies**:
- Contentlayer TOC data (`post.toc`)
- Scroll spy logic (detect active heading)
- Responsive: Hide on mobile, sticky on desktop
**Design Tokens**:
- Active heading: primary color
- Hover state: secondary color
- Typography: body2 variant

#### 2. ArticleCard
**Purpose**: Stunning blog post preview
**Location**: `/components/new/ArticleCard.tsx`
**Props**:
```typescript
interface ArticleCardProps {
  title: string
  summary?: string
  date: string
  tags?: string[]
  image?: string
  slug: string
  readingTime?: { text: string }
}
```
**Dependencies**:
- Framer Motion for interactions
- Image component for optimized loading
- Link component for navigation
**Variants**:
- Default: Standard card with image
- Featured: Larger, prominent placement
- Compact: List view variant
**Design Tokens**:
- Card elevation: shadow-2xl
- Gradient overlays: custom gradients
- Border radius: rounded-2xl
- Hover transform: scale(1.02)

#### 3. QuickPublishForm
**Purpose**: Admin interface for MDX creation
**Location**: `/app/admin/quick-publish/page.tsx`
**Form Fields**:
- title (required), date (default: now), draft (boolean)
- tags (multi-select), authors (multi-select)
- summary (textarea), layout (dropdown)
- body (MDX editor with preview)
**Dependencies**:
- MUI form components
- MDX preview renderer
- File system API for saving
**Security**:
- Authentication required (TBD)
- Server action for file write

### Components to Modify

#### 1. PostLayout.tsx
**Changes**:
- Add `<OnThisPageNav>` to sidebar
- Position: Above tags section
- Conditional: Only show if TOC has 2+ headings

#### 2. ListLayout.tsx
**Changes**:
- Replace current post preview with `<ArticleCard>`
- Add grid layout for cards
- Responsive: 1 col mobile, 2 col tablet, 3 col desktop

#### 3. app/layout.tsx
**Changes**:
- Integrate new theme system
- Update font loading for Rethink Sans
- Remove commented font variables (lines 8-12 in tailwind.config)

---

## Known Constraints & Conflicts

### Constraint 1: Tailwind v4 + MUI v7 Coexistence
**Challenge**: CSS specificity conflicts, duplicate tokens
**Current Mitigation**:
- MUI uses emotion for CSS-in-JS
- Tailwind uses PostCSS layer system
- CSS variables bridge the gap (`--mui-palette-primary-main`)
**Resolution Strategy**:
- Ensure new components use Tailwind utilities FIRST
- Override with MUI components only when necessary
- Document CSS cascade order in style guide

### Constraint 2: Font Configuration Duplication
**Conflict**:
- Fonts loaded in `app/layout.tsx` via next/font/google
- Typography system in `customTheme/typography.ts` uses different fonts
- Tailwind config has commented-out font families
**Resolution Required**:
- Audit all font usage across codebase
- Consolidate to single source of truth
- Update CSS variables to match next/font variables

### Constraint 3: Existing Theme System
**Challenge**: 50+ files in `/customTheme/` directory
**Risk**: Regression in existing components during replacement
**Mitigation**:
- Incremental replacement, not big-bang rewrite
- Maintain backwards compatibility during transition
- Comprehensive visual regression testing (Storybook)

### Constraint 4: Content Build Performance
**Issue**: Contentlayer rebuild on every MDX change
**Impact**: Slow local development for quick-publish feature
**Workaround**:
- Quick-publish saves to `/data/blog/` (triggers rebuild)
- Consider preview mode without full rebuild
- Use Contentlayer's watch mode effectively

---

## Integration Requirements

### Integration 1: Contentlayer TOC
**What**: Extract table of contents for OnThisPageNav
**How**:
```typescript
import { allBlogs } from 'contentlayer/generated'
const post = allBlogs.find(p => p.slug === params.slug)
const toc = post.toc // Array<{ url, depth, value }>
```
**Consumers**: PostLayout, PostSimple, PostBanner
**Data Shape**:
```json
[
  { "url": "#introduction", "depth": 2, "value": "Introduction" },
  { "url": "#getting-started", "depth": 2, "value": "Getting Started" }
]
```

### Integration 2: MDX Component Mapping
**What**: Register new components for use in MDX files
**How**: Update `/components/MDXComponents.tsx`
```typescript
export const components: MDXComponents = {
  // Existing components...
  ArticleCard,
  OnThisPageNav,
}
```
**Usage in MDX**:
```mdx
<ArticleCard
  title="My Post"
  slug="/blog/my-post"
  summary="A great post"
/>
```

### Integration 3: Pliny Search & Analytics
**What**: Ensure new components tracked and searchable
**Search**: Kbar uses `/public/search.json` (auto-generated)
**Analytics**: Umami tracks page views (no component changes needed)
**Concern**: ArticleCard links must use proper Link component for tracking

### Integration 4: MUI + Tailwind Theming
**Current Pattern**:
```typescript
// MUI ThemeProvider wraps app
<ThemeProviders>
  <Analytics />
  <SearchProvider>
    {children}
  </SearchProvider>
</ThemeProviders>
```
**CSS Variables Bridge**:
- MUI exposes: `--mui-palette-primary-main`, etc.
- Tailwind can reference: `bg-[--mui-palette-primary-main]`
- New theme must maintain this pattern

**Color Scheme Sync**:
- MUI: `useColorScheme()` hook
- next-themes: `useTheme()` hook
- ThemeProviders.tsx coordinates both (currently)

---

## File Tracking

### Files to Create
1. `/components/new/OnThisPageNav.tsx`
2. `/components/new/ArticleCard.tsx`
3. `/app/admin/quick-publish/page.tsx`
4. `/app/admin/quick-publish/layout.tsx` (admin layout)
5. `/lib/admin/saveMdx.ts` (server action for file write)
6. `/components/new/OnThisPageNav.stories.tsx` (Storybook)
7. `/components/new/ArticleCard.stories.tsx` (Storybook)

### Files to Modify
1. `/layouts/PostLayout.tsx` (add OnThisPageNav)
2. `/layouts/PostSimple.tsx` (add OnThisPageNav)
3. `/layouts/PostBanner.tsx` (add OnThisPageNav)
4. `/layouts/ListLayout.tsx` (use ArticleCard)
5. `/layouts/ListLayoutWithTags.tsx` (use ArticleCard)
6. `/app/page.tsx` (featured ArticleCards)
7. `/components/MDXComponents.tsx` (register new components)
8. `/tailwind.config.js` (uncomment fonts, add new tokens)
9. `/customTheme/typography.ts` (integrate Rethink Sans)
10. `/app/layout.tsx` (load Rethink Sans font)

### Files to Audit (Theme Replacement)
1. All files in `/customTheme/` (50+ files)
2. `/app/theme-providers.tsx` (current theme wrapper)
3. `/css/tailwind.css` (base styles)
4. `/app/globals.css` (global overrides)

---

## Design Tokens to Define

### Color Palette
**Primary**:
- `--color-primary-500`: #[treegreen from colorGuide]
- `--color-primary-600`: #[deepgreen]
- `--color-primary-400`: #[applegreen]

**Secondary**:
- `--color-secondary-500`: #[walnut]
- `--color-secondary-600`: #[mud]

**Semantic Colors**:
- Error, Warning, Info, Success (already defined in MUI palette)

**Surface Colors**:
- `--surface-default`: white (light), #back (dark)
- `--surface-elevated`: paper token
- `--surface-overlay`: glass effect (rgba)

### Typography Scale
**Font Families**:
- `--font-body`: Rethink Sans, system-ui, sans-serif
- `--font-heading`: Rokkitt, serif
- `--font-mono`: Victor Mono NFM, monospace

**Font Sizes** (Tailwind scale):
- xs, sm, base, lg, xl, 2xl, 3xl, 4xl, 5xl, 6xl

**Font Weights**:
- light (300), regular (400), medium (500), bold (700)

### Spacing Scale
**Tailwind Default**: 0.25rem increments
- 0, 1, 2, 4, 6, 8, 12, 16, 20, 24, 32, 40, 48, 64, 80, 96

**Component-Specific**:
- Card padding: p-6 (24px)
- Section gaps: gap-8 (32px)
- Page margins: mx-auto max-w-7xl px-4

### Elevation (Shadows)
**Levels**:
- sm: subtle border-like shadow
- md: standard card elevation
- lg: modal/drawer elevation
- xl: toast/notification elevation
- 2xl: article card hover state

**Glass Effect**:
```css
backdrop-blur-sm bg-white/10 border border-white/20
```

### Border Radius
**Scale**:
- sm: 0.25rem
- DEFAULT: 0.5rem
- lg: 1rem
- xl: 1.5rem
- 2xl: 2rem
- full: 9999px

### Transitions
**Durations**:
- fast: 150ms
- normal: 300ms
- slow: 500ms

**Easings**:
- ease-in-out (default)
- ease-spring (framer-motion)

---

## Risk Assessment

### Risk 1: Theme System Regression
**Probability**: Medium
**Impact**: High (visual breaks across site)
**Mitigation**:
- Storybook for all components
- Visual regression testing
- Gradual rollout (feature flag)
- Maintain old theme in parallel during transition

### Risk 2: Build Performance Degradation
**Probability**: Medium
**Impact**: Medium (slow dev experience)
**Mitigation**:
- Monitor contentlayer rebuild times
- Optimize MDX processing pipeline
- Use incremental builds where possible

### Risk 3: Font Loading Performance
**Probability**: Low
**Impact**: Medium (FOUT/FOIT)
**Mitigation**:
- Use next/font with display: swap
- Preload critical fonts
- Font subsetting for smaller payloads

### Risk 4: Accessibility Regression
**Probability**: Low
**Impact**: High (compliance issues)
**Mitigation**:
- Run axe DevTools on all new components
- Keyboard navigation testing
- Screen reader testing
- Color contrast validation (WCAG AA)

---

## Success Metrics

### Mission Objective 1: OnThisPageNav
- [ ] Component renders on all blog posts
- [ ] Scroll spy accurately highlights current section
- [ ] Keyboard accessible (tab navigation)
- [ ] Mobile: Hidden or collapsible
- [ ] Performance: No layout shift on load

### Mission Objective 2: ArticleCard
- [ ] 90+ Lighthouse performance score maintained
- [ ] Smooth animations (60fps)
- [ ] Image optimization (WebP, lazy loading)
- [ ] Hover states feel responsive (<100ms)
- [ ] Cards render correctly in all list contexts

### Mission Objective 3: Theme Replacement
- [ ] 100% visual parity with old theme (or intentional improvements)
- [ ] CSS bundle size reduction (target: 20% smaller)
- [ ] No runtime CSS-in-JS (prefer compiled CSS)
- [ ] Dark mode seamless
- [ ] All MUI components still work

### Mission Objective 4: Font Integration
- [ ] Fonts load on first paint (no FOIT)
- [ ] Typography hierarchy clear and readable
- [ ] Code blocks use Victor Mono consistently
- [ ] Headings use Rokkitt appropriately
- [ ] Body text uses Rethink Sans

### Mission Objective 5: Quick-Publish
- [ ] Form validation prevents invalid MDX
- [ ] Preview renders correctly
- [ ] Saved files trigger contentlayer rebuild
- [ ] Files saved with correct frontmatter format
- [ ] Authentication protects endpoint

---

## Next Actions

### Immediate (Context Manager Complete)
1. Share this document with team agents
2. Assign ownership:
   - Component Builder: OnThisPageNav + ArticleCard
   - Theme Engineer: Theme system audit + Rethink Sans integration
   - UX Designer: Design specs for ArticleCard variants
   - QA/Tester: Setup visual regression suite

### Phase 1 (Week 1)
1. Component Builder: Build OnThisPageNav (Storybook first)
2. Theme Engineer: Integrate Rethink Sans font
3. UX Designer: Design ArticleCard variants (3-5 options)
4. QA/Tester: Establish baseline Storybook snapshots

### Phase 2 (Week 2)
1. Component Builder: Build ArticleCard (approved variant)
2. Theme Engineer: Begin theme system consolidation
3. Integration: Add components to layouts
4. QA/Tester: Accessibility audit of new components

### Phase 3 (Week 3)
1. Theme Engineer: Complete theme replacement
2. Component Builder: Quick-publish admin page
3. QA/Tester: Full regression testing
4. Documentation: Update component library docs

---

## Collaboration Protocol

### Decision Authority
- **UX Designer**: Final say on visual design, animations
- **Theme Engineer**: Final say on CSS architecture, token naming
- **Component Builder**: Final say on component API design
- **Context Manager**: Escalation for scope/priority conflicts

### Communication Channels
- **Daily Standup**: Status updates (simulated via context sync)
- **Design Review**: UX Designer presents, team provides feedback
- **Code Review**: All PRs require 1 approval
- **Blocker Resolution**: Context Manager coordinates

### Definition of Done
For each component:
- [ ] Functional implementation complete
- [ ] Storybook stories written
- [ ] Accessibility requirements met (WCAG AA)
- [ ] Unit tests passing (if applicable)
- [ ] Visual regression tests passing
- [ ] Documentation updated
- [ ] Approved in design review
- [ ] Integrated into relevant layouts
- [ ] Performance benchmarks met

---

## Appendix: Key File Paths

### Core Configuration
- Site metadata: `/data/siteMetadata.js`
- Contentlayer config: `/contentlayer.config.ts`
- Tailwind config: `/tailwind.config.js`
- Next.js config: `/next.config.ts`
- Package.json: `/package.json`

### Styling
- Tailwind entry: `/css/tailwind.css`
- Global styles: `/app/globals.css`
- Prism theme: `/css/prism.css`
- MUI theme: `/customTheme/index.ts`

### Components
- MDX components: `/components/MDXComponents.tsx`
- Interactive code: `/components/new/`
- UI primitives: `/components/ui/`
- Layout templates: `/layouts/`

### Content
- Blog posts: `/data/blog/`
- Authors: `/data/authors/`
- Generated: `/.contentlayer/generated/`

### App Structure
- Root layout: `/app/layout.tsx`
- Home page: `/app/page.tsx`
- Blog routes: `/app/blog/[...slug]/page.tsx`
- Tag routes: `/app/tags/[tag]/page.tsx`

---

**Status**: Mission state initialized. Ready for agent assignment and phase 1 execution.
**Last Updated**: 2026-01-20
**Context Manager**: AI Context Manager Agent