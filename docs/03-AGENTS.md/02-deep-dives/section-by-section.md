# Section-by-Section Guide

> Exhaustive reference for every recommended section in an AGENTS.md file.

---

## Table of Sections

| # | Section | Required? | Purpose |
|---|---------|-----------|---------|
| 1 | Setup / Commands | ⭐ Strongly recommended | How to build, test, and run the project |
| 2 | Stack / Technology | ⭐ Strongly recommended | Frameworks, languages, and versions |
| 3 | Project Structure | Recommended | Where important directories live |
| 4 | Conventions | ⭐ Strongly recommended | Coding rules and patterns |
| 5 | Boundaries | ⭐ Strongly recommended | What agents must NEVER do |
| 6 | Testing | Recommended | How to write and run tests |
| 7 | Code Patterns | Optional | Preferred architectural patterns with examples |
| 8 | Git Workflow | Optional | Branch, commit, and PR conventions |
| 9 | Architecture Notes | Optional | Key design decisions to respect |
| 10 | Safety / Compliance | Conditional | For regulated or security-sensitive projects |

---

## 1. Setup / Commands

**Purpose**: Give the agent every command it needs to build, test, lint, and run the project.

### Template

```markdown
## Setup
- Package manager: pnpm (never npm or yarn)
- Install: `pnpm install`
- Dev: `pnpm dev`
- Build: `pnpm build`
- Test (all): `pnpm test`
- Test (single): `pnpm test -- path/to/file.test.ts`
- Lint: `pnpm lint`
- Format: `pnpm format`
- Type check: `pnpm typecheck`
- DB migrate: `pnpm db:migrate`
- DB seed: `pnpm db:seed`
```

### What Makes It Great

- **Exact commands with flags** — `pnpm test -- --reporter=verbose`, not "run tests"
- **Single-file test command** — agents need this for focused testing
- **Explicit package manager** — prevents `npm install` when you use pnpm
- **All common operations** — build, test, lint, format, type check, DB

> For anti-patterns, see [Common Mistakes](../04-anti-patterns/common-mistakes.md).

---

## 2. Stack / Technology

**Purpose**: Tell the agent exactly what technologies and versions are in use.

### Template

```markdown
## Stack
TypeScript 5.4, React 18, Next.js 15 (App Router), Tailwind CSS 3.4,
Prisma 5.x with PostgreSQL, NextAuth.js 5, Vitest + RTL, Playwright.

NOT used: Redux, Webpack, Jest, Enzyme, CSS Modules.
```

### What Makes It Great

- **Single concise line** — not a verbose bulleted list
- **Versions included** — prevents agents from generating deprecated APIs
- **Negative list** — explicitly excludes similar technologies that agents might default to

---

## 3. Project Structure

**Purpose**: Map the codebase so agents know where to put things.

### Template

```markdown
## Structure
src/app/         — Next.js App Router (pages, layouts, API routes)
src/components/  — Reusable React components
src/lib/         — Business logic, utilities, helpers
src/server/      — Server-only code (DB queries, auth logic)
src/types/       — Shared TypeScript types and interfaces
prisma/          — Database schema and migrations
e2e/             — Playwright E2E tests
scripts/         — Dev tooling and automation scripts
```

### What Makes It Great

- **Top-level directories only** — doesn't list individual files
- **Purpose annotations** — explains what goes in each directory
- **Stays current** — easy to update, hard to go stale

---

## 4. Conventions

**Purpose**: Define the coding rules that every change must follow.

### Template

```markdown
## Conventions
- TypeScript strict mode; never use `any` or type assertions
- Named exports only (default exports only for Next.js pages/layouts)
- Functional components with explicit `Props` interface
- Server Components by default; `'use client'` only when required
- All API input validated with Zod schemas
- Errors: custom `AppError` class, never raw `throw new Error()`
- async/await only; never `.then()` chains
- Imports: use `@/` path alias for src/
```

### The "Sweet Spot": 5-10 Rules

| Number of Rules | Effect |
|----------------|--------|
| 1-3 | Too few — misses important patterns |
| 5-10 | ✅ Sweet spot — focused and followable |
| 10-15 | Getting diluted — prioritize |
| 20+ | Agent ignores most of them |

---

## 5. Boundaries

**Purpose**: Define hard lines that agents must NEVER cross.

### Template

```markdown
## Boundaries
- NEVER commit .env files, secrets, or API keys
- NEVER modify files in legacy/ or generated/
- NEVER modify existing Prisma migration files
- NEVER delete database columns (deprecate first, delete in next release)
- NEVER disable ESLint rules with inline comments
- NEVER use console.log (use src/lib/logger instead)
- NEVER add dependencies without noting in PR description
- NEVER use eval(), Function(), or dynamic code execution
```

### Why "NEVER" Not "Don't"

Testing shows that agents comply more consistently with "NEVER" than "don't" or "avoid":

| Phrasing | Compliance Rate |
|----------|----------------|
| "NEVER modify X" | ~90% |
| "Don't modify X" | ~75% |
| "Avoid modifying X" | ~60% |
| "Try not to modify X" | ~40% |
| "We prefer you don't modify X" | ~25% |

Use the strongest language for the strongest constraints.

---

## 6. Testing

**Purpose**: How to write tests, what framework to use, and what coverage is expected.

### Template

```markdown
## Testing
- Framework: Vitest + React Testing Library
- E2E: Playwright
- Pattern: Arrange → Act → Assert
- Naming: `it('should [behavior] when [condition]')`
- Mock external services, never real API calls in tests
- Coverage: 80% for src/lib/ and src/server/
- Run tests after every code change: `pnpm test`
```

---

## 7. Code Patterns

**Purpose**: Show the agent what a correctly structured piece of code looks like.

### Template

```markdown
## Code Patterns

### API Route
```typescript
export async function POST(req: Request) {
  const body = await req.json();
  const input = CreateUserSchema.parse(body);
  const user = await userService.create(input);
  return Response.json(user, { status: 201 });
}
```

### React Component
```typescript
interface ButtonProps {
  label: string;
  onClick: () => void;
  variant?: 'primary' | 'secondary';
}

export function Button({ label, onClick, variant = 'primary' }: ButtonProps) {
  return (
    <button className={styles[variant]} onClick={onClick}>
      {label}
    </button>
  );
}
```
```

### Why This Works

Code examples are unambiguous. One example communicates:
- Naming conventions
- Import patterns
- Type definitions
- Function signatures
- Error handling style
- All in ~10 lines

---

## 8. Git Workflow

**Purpose**: How to branch, commit, and create PRs.

### Template

```markdown
## Git Workflow
- Branch: `feat/description`, `fix/description`, `chore/description`
- Commits: conventional commits (feat:, fix:, docs:, test:, chore:)
- PR target: `main`
- PR title: conventional commit format
- One logical change per commit
- Squash merge preferred
```

---

## 9. Architecture Notes

**Purpose**: Key design decisions that the agent must respect (not a full architecture document).

### Template

```markdown
## Architecture
- Monolith-first: everything runs in one Next.js app
- API routes handle auth → validation → business logic → response
- Database accessed only through Prisma service layer (never raw SQL)
- Events via EventEmitter for cross-module communication
- Feature flags via LaunchDarkly (check before using new features)
```

---

## 10. Safety / Compliance

**Purpose**: For projects with regulatory, security, or compliance requirements.

### Template

```markdown
## Safety & Compliance
- HIPAA: never log PII (names, emails, health data)
- All user input sanitized before database storage
- SQL injection prevention: use parameterized queries only
- XSS prevention: never use dangerouslySetInnerHTML
- Auth required for all /api/ routes except /api/health
- Rate limiting on all public endpoints
- Audit log required for all data mutations
```

---

## Section Priority Matrix

If you can only include N sections, prioritize:

| Lines Available | Include |
|----------------|---------|
| 10 lines | Setup + Stack |
| 30 lines | + Conventions + Boundaries |
| 50 lines | + Testing + Structure |
| 100 lines | + Code Patterns + Git |
| 200 lines | + Architecture + Safety |
