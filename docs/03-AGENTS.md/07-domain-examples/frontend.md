# Frontend AGENTS.md Examples

> Complete AGENTS.md files for React, Next.js, Vue, and frontend-focused projects.

---

## React + Next.js (App Router) + Tailwind

```markdown
# AGENTS.md

## Setup
- Package manager: pnpm (NEVER use npm or yarn)
- Install: `pnpm install`
- Dev: `pnpm dev` (http://localhost:3000)
- Build: `pnpm build`
- Test: `pnpm test`
- Test single: `pnpm test -- path/to/file.test.tsx`
- Lint: `pnpm lint`
- Type check: `pnpm typecheck`

## Stack
TypeScript 5.4, React 18, Next.js 15 (App Router), Tailwind CSS 3.4,
Radix UI primitives, Vitest + React Testing Library, Playwright.

NOT: Redux, CSS Modules, styled-components, Jest, Enzyme.

## Structure
src/app/           — Next.js App Router (pages, layouts, API routes)
src/components/    — Shared React components
src/components/ui/ — Design system primitives (Radix + Tailwind)
src/hooks/         — Custom React hooks
src/lib/           — Utilities, helpers, constants
src/server/        — Server-only code (DB, auth, services)
src/types/         — Shared TypeScript types
public/            — Static assets

## Conventions
- Server Components by default; `'use client'` only for interactivity
- Functional components with explicit `Props` interface
- Named exports only (default exports only for Next.js pages/layouts)
- TypeScript strict mode; never `any` or `as` assertions
- Tailwind for styling; never inline styles or CSS files
- Radix primitives for accessible UI; never build from scratch
- Form validation: Zod + react-hook-form
- Data fetching: server-side in page.tsx or route.ts; never useEffect+fetch

## Component Pattern
```tsx
interface FeatureCardProps {
  title: string;
  description: string;
  onAction: () => void;
}

export function FeatureCard({ title, description, onAction }: FeatureCardProps) {
  return (
    <div className="rounded-lg border p-4 shadow-sm">
      <h3 className="text-lg font-semibold">{title}</h3>
      <p className="mt-2 text-muted-foreground">{description}</p>
      <Button onClick={onAction} className="mt-4">
        Get Started
      </Button>
    </div>
  );
}
```

## Boundaries
- NEVER use `console.log` (use `src/lib/logger`)
- NEVER import from `src/server/` in client components
- NEVER use `useEffect` for data fetching (use server components)
- NEVER modify `src/components/ui/` without design system review
- NEVER commit .env files

## Testing
- Unit: Vitest + React Testing Library
- E2E: Playwright (`pnpm test:e2e`)
- Test behavior, not implementation
- Mock: API calls, auth context
- Name: `it('should [behavior] when [condition]')`
```

---

## Vue 3 + Nuxt 3 + TypeScript

```markdown
# AGENTS.md

## Setup
- Package manager: pnpm
- Install: `pnpm install`
- Dev: `pnpm dev`
- Build: `pnpm build`
- Test: `pnpm test`
- Lint: `pnpm lint`

## Stack
TypeScript 5.x, Vue 3.4 (Composition API), Nuxt 3, UnoCSS,
Pinia stores, VueUse, Vitest + Vue Test Utils.

NOT: Options API, Vuex, Vue 2, Webpack.

## Conventions
- Composition API with `<script setup lang="ts">`
- Define props with `defineProps<T>()` (type-based)
- Pinia for state management (never provide/inject for global state)
- Auto-imports enabled (no manual import for Vue/Nuxt composables)
- UnoCSS for styling

## Boundaries
- NEVER use Options API
- NEVER use Vuex (use Pinia)
- NEVER modify nuxt.config.ts without discussion
- NEVER commit .env files

## Component Pattern
```vue
<script setup lang="ts">
interface Props {
  title: string
  count?: number
}

const props = withDefaults(defineProps<Props>(), {
  count: 0,
})

const emit = defineEmits<{
  update: [value: number]
}>()
</script>

<template>
  <div class="p-4">
    <h2>{{ title }}</h2>
    <span>{{ count }}</span>
  </div>
</template>
```
```

---

## Static Site / Documentation (Astro)

```markdown
# AGENTS.md

## Setup
- Install: `pnpm install`
- Dev: `pnpm dev`
- Build: `pnpm build`
- Preview: `pnpm preview`

## Stack
Astro 4.x, TypeScript, MDX, Tailwind CSS.

## Conventions
- Content in `src/content/` as MDX files
- Pages in `src/pages/` as .astro files
- Components: Astro components preferred; React for interactive islands
- Images optimized with Astro Image

## Structure
src/content/   — MDX blog posts and docs
src/pages/     — Astro page routes
src/components/— Shared components
src/layouts/   — Page layouts
public/        — Static assets

## Boundaries
- NEVER use client-side JavaScript unless absolutely required
- NEVER import large JS libraries (bundle size matters)
- NEVER modify astro.config.mjs without discussion
```

---

## Key Frontend Anti-Patterns to Prevent

| Anti-Pattern | AGENTS.md Rule |
|-------------|----------------|
| useEffect for data fetching | "NEVER use useEffect for fetching; use server components" |
| Prop drilling | "Use Context or Zustand for state shared across 3+ components" |
| Giant components | "Max 150 lines per component; extract sub-components" |
| Inline styles | "NEVER use inline styles; use Tailwind classes" |
| Client-side secrets | "NEVER import env vars without NEXT_PUBLIC_ prefix in client code" |
| Missing accessibility | "All interactive elements must have aria attributes or labels" |

---

*Next: [Backend AGENTS.md](backend.md) →*
