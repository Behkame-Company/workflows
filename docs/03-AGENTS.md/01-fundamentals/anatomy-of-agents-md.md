# Anatomy of an AGENTS.md File

> Structure, recommended sections, and optimal ordering for maximum agent effectiveness.

---

## File Format

AGENTS.md is **plain Markdown**. No frontmatter required, no schema, no special syntax.

```markdown
# AGENTS.md

## Section Heading
- Instruction 1
- Instruction 2

## Another Section
Paragraph of context.
```

---

## Recommended Section Order

AI agents process content **top-down** with strongest attention at the beginning and end. Order sections by impact:

```
1. ## Setup / Commands      ← HIGHEST IMPACT (agents need this first)
2. ## Stack                 ← What technologies are used
3. ## Structure             ← Where things live
4. ## Conventions           ← Coding rules
5. ## Boundaries            ← What agents must NEVER do
6. ## Testing               ← How to verify work
7. ## Git Workflow           ← Commit and PR standards
8. ## Notes                 ← Additional context (LOWEST priority)
```

### Why This Order?

| Position | Attention Level | What to Put Here |
|----------|----------------|------------------|
| First 30% | 🟢 High (primacy) | Commands, stack — agents need these to do anything |
| Middle 40% | 🟡 Medium | Conventions, structure — useful but less critical |
| Last 30% | 🟢 High (recency) | Boundaries, testing — safety-critical reminders |

---

## Section-by-Section Breakdown

### Setup / Commands (Critical)

The single most important section. Without it, agents guess how to build and test your project — and guess wrong.

```markdown
## Setup
- Package manager: pnpm (never use npm or yarn)
- Install: `pnpm install`
- Dev server: `pnpm dev` (http://localhost:3000)
- Tests: `pnpm test`
- Tests (single file): `pnpm test -- path/to/file.test.ts`
- Lint: `pnpm lint`
- Build: `pnpm build`
- DB migrate: `pnpm db:migrate`
- DB seed: `pnpm db:seed`
```

**Rules:**
- Include the **exact command** with flags
- Specify the package manager explicitly
- Add context where helpful ("must pass before PR")
- List single-file test command (agents need this for focused testing)

### Stack

```markdown
## Stack
TypeScript 5.4, React 18, Next.js 15 (App Router), Tailwind CSS 3.4,
Prisma 5.x with PostgreSQL, NextAuth.js 5, Vitest, Playwright.
```

**Rules:**
- Name frameworks with major versions
- Explicitly state what you **don't** use: "NOT: Jest, Webpack, Redux"
- One concise paragraph, not a bulleted list

### Structure

```markdown
## Structure
src/app/         — Next.js App Router pages and layouts
src/components/  — Shared React components (named exports)
src/lib/         — Business logic and utilities
src/server/      — Server-only code (DB queries, auth)
src/types/       — TypeScript type definitions
prisma/          — Database schema and migrations
e2e/             — Playwright end-to-end tests
```

**Rules:**
- Top-level directories only (5-10 lines max)
- Include purpose annotation for each directory
- Don't list individual files — wastes tokens and becomes stale

### Conventions

```markdown
## Conventions
- TypeScript strict mode; never use `any` or type assertions
- Named exports only; no default exports (except Next.js pages)
- Functional components with explicit Props interfaces
- Server Components by default; `'use client'` only for interactivity
- All API input validated with Zod
- `async/await` only; never `.then()` chains
- All monetary values as integers (cents); display via `formatCurrency()`
```

**Rules:**
- 5-10 rules maximum (more dilutes attention)
- Each rule is one line, imperative voice
- Name the specific tool/library to use
- Include the "not" where ambiguity exists

### Boundaries

```markdown
## Boundaries
- NEVER commit secrets, .env files, or API keys
- NEVER modify files in `legacy/` or `generated/`
- NEVER modify existing migration files in `prisma/migrations/`
- NEVER use `console.log` — use the logger from `src/lib/logger`
- NEVER disable ESLint rules with inline comments
- NEVER add dependencies without noting in PR description
- NEVER delete database columns — deprecate first
```

**Rules:**
- Use "NEVER" for absolute rules (stronger signal than "don't")
- Be specific about what and where
- This is the agent's **safety guardrail** — be explicit

### Testing

```markdown
## Testing
- Unit tests: Vitest (`pnpm test`)
- Component tests: Vitest + React Testing Library
- E2E tests: Playwright (`pnpm test:e2e`)
- Pattern: Arrange → Act → Assert
- Naming: `it('should [behavior] when [condition]')`
- Mocks: `vi.mock()` for modules, `vi.fn()` for functions
- Coverage: 80% minimum for src/lib/ and src/server/
```

### Git Workflow

```markdown
## Git Workflow
- Branch naming: `feat/description`, `fix/description`, `chore/description`
- Commit messages: conventional commits (feat:, fix:, docs:, test:)
- PRs target `main` branch
- PRs require passing CI before merge
- One logical change per commit
```

---

## Sizing Guidelines

| File Size | Rating | Notes |
|-----------|--------|-------|
| 20-50 lines | ✅ Ideal | Focused, high signal-to-noise |
| 50-100 lines | ✅ Good | Room for more detail |
| 100-200 lines | ⚠️ Acceptable | Test that all sections are read |
| 200-300 lines | ⛔ Getting long | Consider splitting for monorepos |
| 300+ lines | 🚫 Too long | Agent performance degrades |

**Target: under 200 lines, under 1,000 words.**

---

## Formatting Best Practices

### ✅ Effective: Bullets and Code Blocks

```markdown
## Conventions
- Named exports only
- TypeScript strict mode
- Validate with Zod
```

### ❌ Ineffective: Dense Paragraphs

```markdown
## Conventions
When writing code for this project, we generally prefer to use named exports
rather than default exports, and we try to keep TypeScript in strict mode
whenever possible. For validation, we tend to lean towards using Zod, though
other approaches may be acceptable in some cases.
```

**Why?** Agents parse bullets more reliably than paragraphs. Each bullet becomes a discrete, followable instruction.

---

*Next: [How Agents Read AGENTS.md](how-agents-read.md) →*
