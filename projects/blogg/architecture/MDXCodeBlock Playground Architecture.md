---
title: MDXCodeBlock Playground Architecture
type: reference
permalink: projects/blogg/architecture/mdx-playground
entity_type: reference
status: evergreen
tags:
- project/blogg
- domain/architecture
- domain/mdx
---

# MDXCodeBlock Playground Architecture

Interactive code playground system within MDX blog posts using React Live. Allows readers to view, edit, and execute React components inline.

## Transformation Pipeline

```
MDX sourceCode prop --> prepareLiveCode()
  1. Import Statement Processing (regex --> variable declarations)
  2. TypeScript Type Stripping (generics, annotations, interfaces)
  3. Export Statement Removal
  4. Component Detection (function/arrow function patterns)
  5. Render Call Injection (`render(<ComponentName />)`)
  --> LiveProvider (noInline=true) --> LivePreview + LiveEditor
```

## Component Hierarchy

| Component | Role |
|-----------|------|
| `Pre` | Simple markdown code blocks with copy button |
| `CodeBlock` | Base syntax highlighting (react-syntax-highlighter) |
| `MDXCodeBlock` | Full playground: live editor + preview + action panel |
| `CodeSample` | Example implementation pattern |

## Known Issue: TypeScript Transformation

The TS-to-JS regex stripping can produce syntax errors in React Live's parser (e.g., complex generics like `useRef<Array<HTMLDivElement | null>>`). The preview renders correctly but the error display fires. Needs more robust type stripping or a proper TS transform (e.g., `@babel/plugin-transform-typescript`).

## Key Files

- `components/new/MDXCodeBlock.tsx` — Main playground component
- `components/new/README.md` — Feature documentation (action panel, view modes)

## Relations

- child_of [[Next.js Blog (blogg)]]
- uses [[React Live]]
- related_to [[Contentlayer]]
