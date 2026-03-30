# Domain Guide: Frontend & React

> Instruction patterns for React/Next.js components, styling, and client-side code.

---

## Sample: frontend.instructions.md

```yaml
---
applyTo: "src/components/**/*.{tsx,ts}"
---
```

```markdown
# Frontend Component Instructions

## Components
- Functional components with explicit `Props` interface
- Named exports only (no default exports)
- Component name must match filename: `UserProfile.tsx` → `export function UserProfile()`
- Server Components by default; add `'use client'` only when using hooks, event handlers, or browser APIs

## Styling
- Tailwind CSS classes only; no inline styles, no CSS Modules
- Use `cn()` from `src/lib/utils` for conditional classes
- Responsive design: mobile-first (`sm:`, `md:`, `lg:` breakpoints)
- Dark mode: use `dark:` variant for all colors

## Accessibility
- All interactive elements need `aria-label` or visible label
- Images require `alt` text (empty string for decorative images)
- Use semantic HTML: `<button>` for actions, `<a>` for navigation
- Forms use `<label>` with `htmlFor` attribute
- Focus management for modals and dialogs

## State
- Local state: `useState` / `useReducer`
- Global state: Zustand stores in `src/stores/`
- Server state: TanStack Query with hooks in `src/hooks/queries/`
- URL state: `useSearchParams()` for filter/sort/page

## Patterns
- Extract complex logic into custom hooks (`src/hooks/`)
- Use Next.js `<Image>` for all images (width/height required)
- Use Next.js `<Link>` for internal navigation
- Loading states: use Suspense boundaries, not conditional rendering
- Error states: use Error Boundaries with fallback UI

## Do Not
- Never use `document.querySelector()` or direct DOM manipulation
- Never use `dangerouslySetInnerHTML` without sanitization
- Never mutate state directly (create new references)
- Never import from `services/` or `server/` in client components
```

---

## Key Considerations

### Component Patterns

| Rule | Rationale |
|------|-----------|
| Named exports only | Enables better tree-shaking and IDE navigation |
| Match filename to component | Consistency and discoverability |
| Server Components default | Next.js App Router optimizes for this |
| Extract hooks | Separation of concerns, testability |

### Styling Decisions to Encode

| Decision | Instruction |
|----------|------------|
| CSS framework | "Tailwind CSS only" |
| Conditional classes | "Use `cn()` utility" |
| Theme approach | "Dark mode via `dark:` variant" |
| Responsive strategy | "Mobile-first breakpoints" |

### State Management Map

| State Type | Solution | Instruction |
|-----------|----------|-------------|
| Local UI state | useState | "For component-local state" |
| Complex local state | useReducer | "For multi-field forms or state machines" |
| Global client state | Zustand | "Stores in `src/stores/`" |
| Server/async state | TanStack Query | "Hooks in `src/hooks/queries/`" |
| URL state | useSearchParams | "For filters, sorts, pagination" |

---

## Example: Page-Specific Instructions

```yaml
---
applyTo: "src/app/**/{page,layout,loading,error,not-found}.tsx"
---
```

```markdown
# Next.js Route Files
- Page components may use default exports (Next.js requirement)
- Layouts receive `children` prop; no other data fetching
- Loading files export a skeleton/spinner component
- Error files must be `'use client'` and receive `error` and `reset` props
- Metadata: export `metadata` object or `generateMetadata` function
```
