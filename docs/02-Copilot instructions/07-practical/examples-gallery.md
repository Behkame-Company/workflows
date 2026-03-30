# Real-World Examples Gallery

> 20+ ready-to-use instruction examples organized by project type and domain.

---

## Full copilot-instructions.md Examples

### Example 1: Next.js SaaS Application

```markdown
# Copilot Instructions

## Project
B2B analytics dashboard. Next.js 15 App Router, TypeScript, Tailwind CSS,
Prisma with PostgreSQL, NextAuth.js for authentication.

## Commands
- `pnpm install` — Install dependencies
- `pnpm dev` — Start dev server (localhost:3000)
- `pnpm test` — Run Vitest tests
- `pnpm lint` — ESLint + Prettier
- `pnpm build` — Production build (must pass before merge)
- `pnpm db:migrate` — Run Prisma migrations
- `pnpm db:studio` — Open Prisma Studio

## Structure
src/app/          # Next.js App Router (pages, layouts, API routes)
src/components/   # Shared React components
src/lib/          # Business logic, utilities, database helpers
src/hooks/        # Custom React hooks
src/types/        # TypeScript type definitions
prisma/           # Schema and migrations

## Conventions
- TypeScript strict mode; never use `any`
- Named exports only; no default exports (except Next.js pages)
- Functional components with explicit Props interfaces
- Server Components by default; `'use client'` only when needed
- Validate API input with Zod schemas in `src/lib/validators/`
- Monetary values as integers (cents); display with `formatCurrency()`
- All dates in UTC; display with `formatDate()` using user's timezone

## Do Not
- Never commit .env files or secrets
- Never use console.log — use the logger from `src/lib/logger`
- Never modify prisma/migrations/ manually
- Never bypass auth middleware for API routes
- Never add dependencies without noting in PR description

## Validation
After changes: `pnpm lint && pnpm test && pnpm build`
```

---

### Example 2: Express.js Microservice

```markdown
# Copilot Instructions

## Project
Payment processing microservice. Express.js 4, TypeScript, Prisma, PostgreSQL.
Processes payments via Stripe API. Internal service — not public-facing.

## Commands
- `npm install` — Install dependencies
- `npm run dev` — Start with nodemon (port 3001)
- `npm test` — Run Jest tests
- `npm run lint` — ESLint check
- `npm run build` — Compile TypeScript to dist/

## Structure
src/routes/       # Express route handlers
src/services/     # Business logic (PaymentService, RefundService)
src/middleware/    # Auth, validation, rate limiting, error handling
src/models/       # Prisma model extensions
src/utils/        # Shared utilities
src/types/        # TypeScript types and interfaces

## Conventions
- Route → Middleware → Service → Response (never mix layers)
- All input validated with Zod schemas before reaching services
- Responses: `{ data: T }` or `{ error: { message, code } }`
- Async handlers wrapped with `asyncHandler()` utility
- Stripe API calls isolated in `src/services/stripe.service.ts`
- All monetary amounts in smallest currency unit (cents)

## Do Not
- Never expose Stripe secret keys in responses or logs
- Never process payments without idempotency keys
- Never return raw Stripe errors to callers (wrap in AppError)
- Never modify completed transaction records
```

---

### Example 3: Python FastAPI Backend

```markdown
# Copilot Instructions

## Project
Machine learning inference API. FastAPI 0.110+, Python 3.12,
SQLAlchemy with PostgreSQL, Redis for caching.

## Commands
- `pip install -r requirements.txt` — Install dependencies
- `uvicorn app.main:app --reload` — Start dev server (port 8000)
- `pytest` — Run all tests
- `pytest -x` — Stop on first failure
- `ruff check .` — Lint code
- `mypy app/` — Type checking

## Structure
app/
  api/        # FastAPI route handlers
  core/       # Config, security, dependencies
  models/     # SQLAlchemy models
  schemas/    # Pydantic request/response models
  services/   # Business logic
  ml/         # ML model loading and inference
tests/        # Test files mirroring app/ structure

## Conventions
- Type hints on all function signatures
- Pydantic models for all request/response serialization
- Dependency injection via FastAPI's Depends()
- Async handlers for I/O-bound operations
- Sync handlers for CPU-bound ML inference
- f-strings for string formatting
- snake_case for variables, functions, modules
- PascalCase for classes and Pydantic models

## Do Not
- Never load ML models inside request handlers (use lifespan events)
- Never block the event loop with synchronous I/O
- Never use mutable default arguments
- Never store model files in git (use model registry)
```

---

## Path-Specific Instruction Examples

### React Hooks Directory

```yaml
---
applyTo: "src/hooks/**/*.ts"
---
```

```markdown
# Custom Hook Instructions
- Hook names start with `use` (e.g., `useAuth`, `useDebounce`)
- One hook per file, filename matches hook name
- Return object (not array) for hooks with multiple return values
- Include JSDoc with @example for usage documentation
- Write tests in co-located `*.test.ts` files
- For async hooks, handle loading, error, and data states
```

### GraphQL Schema

```yaml
---
applyTo: "**/*.graphql"
---
```

```markdown
# GraphQL Schema Instructions
- Types use PascalCase: `User`, `ProductOrder`
- Fields use camelCase: `firstName`, `orderTotal`
- Mutations follow verb-noun: `createUser`, `updateProduct`
- All queries require authentication unless marked `@public`
- Pagination uses cursor-based connection pattern
- Deprecations use `@deprecated(reason: "...")` directive
```

### Storybook Stories

```yaml
---
applyTo: "**/*.stories.{tsx,ts}"
---
```

```markdown
# Storybook Instructions
- One story file per component
- Default export: component meta (title, component, argTypes)
- Named exports: individual stories (Default, WithError, Loading, Empty)
- Use `args` for prop variations, not separate renders
- Include decorators for providers (theme, query client)
- Add play functions for interaction testing
```

### Migration Files

```yaml
---
applyTo: "migrations/**"
---
```

```markdown
# Migration File Instructions
- NEVER modify existing migration files
- If a migration is wrong, create a NEW migration to fix it
- Review generated SQL before committing
- Add comments explaining complex operations
```

---

## Prompt File Examples

### Architecture Decision Record

```markdown
---
name: create-adr
description: Create an Architecture Decision Record
---
# Create ADR

Create an Architecture Decision Record in `docs/decisions/NNNN-[title].md`:

## Template
- **Title**: [Short descriptive title]
- **Status**: Proposed
- **Context**: What is the issue that is motivating this decision?
- **Decision**: What is the change that we're proposing?
- **Consequences**: What becomes easier or harder?

Reference existing ADRs in `docs/decisions/` for format consistency.
```

### API Endpoint Generator

```markdown
---
name: create-endpoint
description: Generate a new API endpoint with validation and tests
agent: agent
tools: [search, edit, run]
---
# Create API Endpoint

1. Read existing endpoints in `src/app/api/` for patterns
2. Create the route handler with proper HTTP methods
3. Create a Zod validation schema in `src/validators/`
4. Add authentication check if the endpoint is protected
5. Create test file with success and error cases
6. Run `pnpm test` to verify

## Endpoint Details
[Describe the endpoint: path, method, input, output, auth requirements]
```
