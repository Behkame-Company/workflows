# Real-World AGENTS.md Examples Gallery

> 15+ copy-paste AGENTS.md examples for different project types.

---

## Minimal (10 Lines)

Perfect for getting started. Copy, modify, commit.

```markdown
# AGENTS.md

## Setup
- Install: `npm install`
- Test: `npm test`
- Build: `npm run build`

## Boundaries
- NEVER commit .env files
- NEVER use `any` in TypeScript
```

---

## Standard Web App (50 Lines)

```markdown
# AGENTS.md

## Setup
- Package manager: pnpm (NEVER npm or yarn)
- Install: `pnpm install`
- Dev: `pnpm dev`
- Test: `pnpm test`
- Build: `pnpm build`
- Lint: `pnpm lint`

## Stack
TypeScript 5.4, React 18, Next.js 15 (App Router), Tailwind CSS, Prisma, PostgreSQL.

## Structure
src/app/         — Pages and API routes
src/components/  — React components
src/lib/         — Utilities and helpers
src/server/      — Server-only code
prisma/          — Database schema

## Conventions
- TypeScript strict mode; never `any`
- Named exports only
- Server Components by default
- Validate API input with Zod
- async/await; never .then()

## Boundaries
- NEVER commit .env or secrets
- NEVER modify prisma/migrations/
- NEVER use console.log (use logger)
- NEVER import server code in client components

## Testing
- Vitest + React Testing Library
- Run: `pnpm test`
- All new functions need tests
```

---

## CLI Tool (Node.js)

```markdown
# AGENTS.md

## Setup
- Install: `pnpm install`
- Dev: `pnpm dev` (runs with ts-node)
- Build: `pnpm build` (compiles to dist/)
- Test: `pnpm test`
- Run locally: `pnpm start -- [args]`

## Stack
TypeScript, Node.js 20, Commander.js, Inquirer.js, Vitest.

## Structure
src/
├── cli.ts          — Entry point, command registration
├── commands/       — One file per command
├── lib/            — Shared utilities
└── types/          — TypeScript types

## Conventions
- Each command in its own file under commands/
- Commander.js for argument parsing
- Inquirer.js for interactive prompts
- Exit codes: 0 success, 1 error, 2 invalid args
- Errors: print to stderr, never throw uncaught

## Boundaries
- NEVER use process.exit() in library code (only in cli.ts)
- NEVER use colors/chalk in non-TTY output
- NEVER assume file paths (use path.resolve)
```

---

## Python Data Science / ML

```markdown
# AGENTS.md

## Setup
- Python: 3.12
- Install: `uv sync`
- Test: `uv run pytest -xvs`
- Lint: `uv run ruff check .`
- Notebooks: `uv run jupyter lab`

## Stack
Python 3.12, pandas 2.2, scikit-learn 1.4, pytorch 2.2,
matplotlib, jupyter, pytest.

## Structure
src/data/        — Data loading and preprocessing
src/models/      — ML model definitions and training
src/evaluation/  — Metrics and evaluation scripts
src/utils/       — Shared utilities
notebooks/       — Jupyter notebooks (exploration only)
tests/           — pytest tests
data/            — Data directory (git-ignored)

## Conventions
- Type hints on all functions
- Docstrings (Google style) on all public functions
- Notebooks for exploration only; production code in src/
- Reproducibility: random seeds set, all experiments logged
- Data paths via config, never hardcoded

## Boundaries
- NEVER commit data files (use .gitignore)
- NEVER commit model weights (use DVC or model registry)
- NEVER use global variables for state
- NEVER import from notebooks in source code
```

---

## Open-Source Library

```markdown
# AGENTS.md

## Setup
- Install: `pnpm install`
- Test: `pnpm test`
- Build: `pnpm build`
- Lint: `pnpm lint`
- Docs: `pnpm docs:build`

## Stack
TypeScript 5.4, Vitest, tsup (bundling), typedoc.

## Conventions
- Every public API function must have JSDoc
- Semver: breaking changes = major, features = minor, fixes = patch
- Tree-shakeable: named exports only, no side effects
- Support ESM and CJS (dual package)
- Minimum Node.js version: 18

## Boundaries
- NEVER add runtime dependencies without discussion
- NEVER break existing public API without major version bump
- NEVER use Node.js-specific APIs (must work in browser too)
- NEVER use `any` — all types must be explicit

## Testing
- 95% coverage required for src/
- Test all edge cases and error paths
- Test type inference (compile-time tests with tsd)
```

---

## Monorepo Root (Turborepo)

```markdown
# AGENTS.md

## Monorepo
Turborepo monorepo with pnpm workspaces.
Per-package AGENTS.md in apps/ and packages/.

## Setup
- Install: `pnpm install` (from root)
- Build all: `pnpm turbo build`
- Test all: `pnpm turbo test`
- Test affected: `pnpm turbo test --filter=...[HEAD^]`
- Lint all: `pnpm turbo lint`

## Global Rules
- Package manager: pnpm (NEVER npm or yarn)
- Conventional commits with scope: `feat(web): description`
- Branch: `feat/scope/description`
- PRs target `main`

## Boundaries
- NEVER import between apps/ (shared code goes in packages/)
- NEVER add root-level dependencies
- NEVER commit .env files
- NEVER modify .github/workflows/ without DevOps review
```

---

## Mobile App (React Native / Expo)

```markdown
# AGENTS.md

## Setup
- Install: `pnpm install`
- iOS: `pnpm ios`
- Android: `pnpm android`
- Test: `pnpm test`
- Lint: `pnpm lint`

## Stack
TypeScript, React Native 0.74, Expo SDK 51,
React Navigation 6, Zustand, Vitest.

## Conventions
- Functional components only
- NativeWind for styling (Tailwind-like)
- React Navigation for routing
- Zustand for global state
- Platform-specific code: `.ios.tsx` / `.android.tsx` suffixes

## Boundaries
- NEVER use platform-specific APIs without Platform.OS check
- NEVER use web-only libraries
- NEVER modify app.json without mobile team review
- NEVER commit .env or signing keys
```

---

## Microservice (Event-Driven)

```markdown
# AGENTS.md

## Setup
- Install: `pnpm install`
- Dev: `pnpm dev`
- Test: `pnpm test`
- Build: `pnpm build`

## Stack
TypeScript, Node.js 20, Express, Bull (Redis queues),
PostgreSQL, Prisma, Vitest.

## Architecture
Event-driven microservice. Consumes events from Redis queues,
processes them, and publishes results.

## Conventions
- One handler per event type in src/handlers/
- Idempotent: same event processed twice = same result
- Dead letter queue for failed events
- Structured logging with correlation IDs

## Boundaries
- NEVER call other services synchronously (use events)
- NEVER share database with other services
- NEVER process events without idempotency check
- NEVER commit .env or secrets
```

---

## Using These Examples

1. **Find the closest match** to your project
2. **Copy the template** to your repo root as `AGENTS.md`
3. **Customize**: Update commands, stack, and conventions
4. **Add boundaries**: What must agents NEVER do?
5. **Test**: Have an agent try a task and see if it follows the rules
6. **Iterate**: Add rules when you notice agent mistakes

---

*Next: [Templates Library](templates-library.md) →*
