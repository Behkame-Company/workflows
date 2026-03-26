# copilot-instructions.md Template

> Copy this to `.github/copilot-instructions.md` and customize for your project.

---

```markdown
# Repository Instructions

## Project
<!-- 2-3 sentences: what it is, stack, status -->
[Project name] is a [type of application] built with [primary technologies].
Currently [status: in development / production / maintenance].

## Commands
<!-- List every command the AI might need to run -->
- `npm install` — Install dependencies (always run after pulling changes)
- `npm run dev` — Start development server (http://localhost:3000)
- `npm run build` — Production build (must pass before merging PRs)
- `npm run test` — Run all tests (Jest + React Testing Library)
- `npm run test:e2e` — End-to-end tests (Playwright)
- `npm run lint` — ESLint + Prettier check
- `npm run db:migrate` — Apply database migrations
- `npm run db:seed` — Seed development database

Always run `npm install` before building if dependencies changed.
Always run `npm run lint && npm run test` before considering work complete.

## Structure
<!-- Key directories the AI needs to know about -->
src/
  app/          # Next.js App Router pages and API routes
  components/   # Reusable React components
  lib/          # Utilities, helpers, business logic
  types/        # Shared TypeScript type definitions
  hooks/        # Custom React hooks
  db/           # Database client and queries
tests/
  e2e/          # Playwright end-to-end tests

## Conventions
<!-- 5-10 of your most important rules -->
- TypeScript strict mode; never use `any` type
- Named exports only; no default exports
- Use `async/await`, not `.then()` chains
- Components follow: `ComponentName/index.tsx` + `ComponentName.test.tsx`
- API routes: validate all input with Zod schemas from `src/lib/validators/`
- Use the project logger (`src/lib/logger.ts`), not `console.log`
- All user-facing strings must use the i18n system
- Prefer composition over inheritance

## Do Not
<!-- Explicit boundaries -->
- Never commit secrets or API keys; use environment variables via `.env.local`
- Never modify files in `legacy/` or `generated/`
- Never disable TypeScript errors with `@ts-ignore` or `@ts-expect-error`
- Never add new dependencies without discussing with the team
- Never use `console.log` in production code

## Testing
- Every new module must have a co-located test file
- Test behavior, not implementation details
- Use `userEvent` over `fireEvent` for component tests
- Mock external services; never make real API calls in tests

## Validation
After making changes:
1. `npm run lint` — fix all errors
2. `npm run test` — all tests must pass
3. `npm run build` — must compile without errors

## Project Context
<!-- Optional: link to memory bank -->
For current work status and architecture decisions, refer to `memory-bank/activeContext.md`.
```
