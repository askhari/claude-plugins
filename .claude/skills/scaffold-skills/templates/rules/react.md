---
applies_to:
  - "**/*.tsx"
  - "**/*.jsx"
  - "**/components/**/*.ts"
  - "**/components/**/*.js"
  - "**/hooks/**/*.ts"
---

# React conventions for this project

## Component model

- Functional components only. No `class` components in new code.
- TypeScript strict mode. `any` is forbidden in component code; use `unknown` and narrow.
- Props typed via interface or type alias, exported alongside the component if consumers need to re-use them.
- One component per file. Filename matches the export: `UserCard.tsx` exports `UserCard`.

## Hooks

- Hooks run unconditionally — no `if`/`for` around a hook call.
- Effect dependencies are explicit and exhaustive. Don't disable the lint rule to "fix" a re-render bug; fix the dependency.
- Custom hooks live in `hooks/` and start with `use`. They return objects with named fields, not tuples (tuples don't scale past two values).
- `useMemo` / `useCallback` only when there's a measurable problem. Premature memoization is worse than no memoization.

## State

- Local state in `useState` first. Lift only when two siblings need it.
- Server state via TanStack Query / SWR — not `useEffect` + `fetch`. Cache invalidation is a solved problem; don't redo it.
- Global client state via Zustand / Jotai / Context — Redux only if it's already in the project.

## File layout

```
src/
├── components/
│   ├── <ComponentName>/
│   │   ├── <ComponentName>.tsx
│   │   ├── <ComponentName>.test.tsx
│   │   └── index.ts
├── hooks/
├── lib/                # framework-agnostic utilities
├── routes/ or pages/   # depending on framework
└── styles/
```

## Styling

- One styling approach per project. Don't mix CSS Modules + Tailwind + styled-components in new code.
- Tailwind: extract repeated class strings into components, not into `@apply`.
- CSS-in-JS: keep style definitions next to the component, not in shared files.

## Testing

- React Testing Library, not Enzyme.
- Query by accessible role (`getByRole`) over `getByTestId`. Test IDs are a last resort.
- No snapshot tests for components with non-trivial logic — they ratify bugs.
- Mock at the network boundary (MSW), not at the component or hook level.

## Performance

- `key` props on lists must be stable IDs from the data, not array indices (indices are correct only when the list is append-only and unsorted).
- Code-split at route boundaries with `React.lazy` + `Suspense`. Don't split tiny components.
- Images: explicit `width` and `height` (or `aspect-ratio` CSS) to prevent layout shift.

## Before declaring done

- [ ] `tsc --noEmit` (or `vue-tsc --noEmit` for Vue projects)
- [ ] `eslint .` clean — no `// eslint-disable-next-line` without a comment
- [ ] `prettier --check .` (the hook auto-formats on save)
- [ ] Component tests for non-trivial behavior

## Anti-patterns

- `useEffect` to derive state from props. Compute it directly during render.
- Hidden re-render triggers: passing inline object/array literals as props to memoized children.
- Reaching into the DOM with `document.querySelector`. Use refs.
- `dangerouslySetInnerHTML` on anything that hasn't been sanitized at the source.
