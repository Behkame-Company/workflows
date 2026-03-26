# Repository-Wide Instructions: copilot-instructions.md

> The single most impactful file for Copilot customization.

---

## Overview

| Property | Value |
|----------|-------|
| **File** | `.github/copilot-instructions.md` |
| **Scope** | Entire repository |
| **Auto-loaded** | Yes — every Copilot interaction in this repo |
| **Supported by** | Copilot Chat, Coding Agent, Code Review |
| **Priority** | Below personal, above organization |

---

## When to Use

✅ **Use for:**
- Build/test/lint commands (the #1 most impactful thing)
- Stack description (framework, language, major dependencies)
- Project structure overview
- Universal coding conventions
- Global "do not" rules

❌ **Don't use for:**
- Rules that only apply to frontend or backend (use path-specific)
- One-off task workflows (use prompt files)
- Cross-tool compatibility (use AGENTS.md)
- Massive documentation dumps (exceeds context budget)

---

## Recommended Structure

```markdown
# Project Instructions for Copilot

## Project
[2-3 sentence project description with stack]

## Commands
- `[install command]` — Install dependencies
- `[dev command]` — Start development server
- `[test command]` — Run tests
- `[lint command]` — Check code quality
- `[build command]` — Production build

## Structure
[5-10 line directory overview]

## Conventions
- [Rule 1]
- [Rule 2]
- [Rule 3 — keep to 5-10 rules]

## Do Not
- Never [absolute boundary 1]
- Never [absolute boundary 2]
- Never [absolute boundary 3]

## Validation
After changes:
1. `[lint command]`
2. `[test command]`
3. `[build command]`
```

---

## Real-World Example

```markdown
# Copilot Instructions

## Project
E-commerce checkout service. Next.js 15 with App Router, TypeScript,
Tailwind CSS, Prisma ORM with PostgreSQL.

## Commands
- `pnpm install` — Install dependencies
- `pnpm dev` — Start dev server at http://localhost:3000
- `pnpm test` — Run Jest tests
- `pnpm lint` — Run ESLint + Prettier
- `pnpm build` — Production build (must pass before merge)
- `pnpm db:push` — Push Prisma schema changes to dev DB
- `pnpm db:migrate` — Create and apply migration

## Structure
src/
  app/           # Next.js App Router pages and layouts
  components/    # Shared React components
  lib/           # Business logic and utilities
  server/        # Server-only code (API routes, DB queries)
  types/         # TypeScript type definitions
prisma/          # Database schema and migrations
e2e/             # Playwright end-to-end tests

## Conventions
- TypeScript strict mode; never use `any` or `as` type assertions
- Named exports only (no default exports)
- Functional components with explicit Props interfaces
- Server Components by default; add 'use client' only when needed
- Validate all API input with Zod schemas in `src/server/validators/`
- Use `async/await`; never `.then()` chains
- All monetary values stored as integers (cents), displayed with `formatCurrency()`
- Errors returned as `{ success: false, error: string }`, never thrown in API routes

## Do Not
- Never commit .env files or secrets
- Never modify `prisma/migrations/` by hand — use `pnpm db:migrate`
- Never use `console.log` — use the `logger` from `src/lib/logger.ts`
- Never add dependencies without noting in the PR description
- Never disable ESLint rules with inline comments

## Testing
- Unit tests co-located: `Component.test.tsx` next to `Component.tsx`
- Integration tests in `__tests__/` directories
- E2E tests in `e2e/` using Playwright
- Minimum coverage: 80% for business logic in `src/lib/`

## Validation
After making changes:
1. `pnpm lint` — Fix all lint errors
2. `pnpm test` — All tests pass
3. `pnpm build` — Compiles without errors
4. If DB changes: `pnpm db:push` — Schema syncs
```

---

## Sizing Guidelines

| Size | Rating | Notes |
|------|--------|-------|
| 200-500 words | ✅ Ideal | Fits comfortably in context |
| 500-1,000 words | ⚠️ Acceptable | Test that all sections are read |
| 1,000-2,000 words | ⛔ Too long | Middle content likely ignored |
| 2,000+ words | 🚫 Counterproductive | Move content to path-specific files |

### How to Stay Under Budget

If your instructions keep growing, extract domain-specific rules:

```
Before (all in copilot-instructions.md):
  - 15 universal rules
  - 10 frontend rules
  - 10 backend rules
  - 5 database rules
  = 40 rules (too many)

After:
  copilot-instructions.md: 15 universal rules
  frontend.instructions.md: 10 frontend rules
  backend.instructions.md: 10 backend rules
  database.instructions.md: 5 database rules
  = 15 rules in any given context (much better)
```

---

## Common Patterns

### Pattern: Build Commands First

The most impactful thing you can include is build/test commands. Without these, the Coding Agent will **guess** how to run your project (and guess wrong).

### Pattern: Explicit Stack Declaration

```markdown
## Project
Stack: React 18, TypeScript 5.3, Vite 5, Vitest, Zustand
NOT: Angular, Webpack, Jest, Redux (do not suggest these)
```

Explicitly naming what you **don't** use prevents Copilot from suggesting wrong frameworks.

### Pattern: Monetary/Measurement Convention

```markdown
## Data Conventions
- All prices in cents (integer), displayed via `formatPrice()` in `src/lib/format.ts`
- All dates in ISO 8601 UTC, displayed via `formatDate()` in `src/lib/format.ts`
- All weights in grams (integer)
```

Domain-specific data conventions prevent subtle bugs.

### Pattern: Error Handling Convention

```markdown
## Error Handling
API routes return `{ success: false, error: string, code?: string }`
Client code checks `result.success` before accessing data.
Never throw errors in API routes — catch and return error responses.
```

---

## Gotchas

| Gotcha | Details |
|--------|---------|
| File must be in `.github/` folder | Not the repo root |
| Case-sensitive filename | Must be `copilot-instructions.md` exactly |
| Not loaded for inline completions | Only affects Chat, Coding Agent, Code Review |
| Content after ~4,000 chars truncated | For code review specifically |
| Empty or commented-out instructions | Copilot ignores them silently |

---

*Next: [Path-Specific Instructions](path-specific-instructions.md) →*
