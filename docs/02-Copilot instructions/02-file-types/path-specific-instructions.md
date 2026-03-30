# Path-Specific Instructions: .instructions.md

> Scoped rules for specific files, folders, languages, or domains.

---

## Overview

| Property | Value |
|----------|-------|
| **Location** | `.github/instructions/NAME.instructions.md` |
| **Scope** | Files matching the `applyTo` frontmatter glob |
| **Auto-loaded** | Yes — when working on matching files |
| **Format** | YAML frontmatter + Markdown body |
| **Supported by** | Copilot Chat, Coding Agent, Code Review |

---

## File Format

```markdown
---
applyTo: "src/components/**/*.tsx"
---
# Frontend Component Instructions

- Use functional components with TypeScript Props interface
- Tailwind CSS for all styling; no inline styles
- Every component must have a corresponding `.test.tsx` file
```

### Frontmatter Fields

| Field | Required | Description |
|-------|----------|-------------|
| `applyTo` | ✅ Yes | Glob pattern for matching files |
| `excludeAgent` | ❌ No | Agents to exclude: `"code-review"` or `"coding-agent"` |

---

## The applyTo Glob Syntax

The `applyTo` field uses standard glob patterns:

| Pattern | Matches |
|---------|---------|
| `"**/*.ts"` | All TypeScript files in any directory |
| `"src/components/**"` | Everything under src/components/ |
| `"*.test.ts"` | Test files in root only |
| `"**/*.test.{ts,tsx}"` | All TypeScript test files |
| `"prisma/**"` | Everything in prisma/ directory |
| `"src/api/**/*.ts"` | TypeScript files in src/api/ tree |
| `"**/*.{css,scss}"` | All CSS and SCSS files |
| `"Dockerfile*"` | Dockerfile and Dockerfile.dev etc. |

### Multi-Pattern Support

You can target multiple patterns:

```yaml
---
applyTo: "**/*.{test.ts,test.tsx,spec.ts,spec.tsx}"
---
```

---

## Practical Examples

### Frontend Components

```markdown
---
applyTo: "src/components/**/*.tsx"
---
# Component Guidelines

- Functional components with explicit `Props` interface
- No default exports — use named exports only
- Tailwind CSS classes only; no CSS Modules or inline styles
- Use `cn()` from `src/lib/utils` for conditional classes
- Extract complex logic into custom hooks in `src/hooks/`
- Accessibility: all interactive elements need aria-labels
- Images: use Next.js `<Image>` component with width/height
```

### API Routes / Backend

```markdown
---
applyTo: "src/app/api/**/*.ts"
---
# API Route Guidelines

- Validate all request input with Zod
- Return typed responses: `{ data: T }` or `{ error: string }`
- Use proper HTTP status codes (200, 201, 400, 401, 404, 500)
- Wrap all handlers in try/catch with error logging
- Never expose internal errors to clients
- Rate limiting via `src/lib/rate-limit.ts` for public endpoints
- Auth check via `getServerSession()` for protected routes
```

### Database Layer

```markdown
---
applyTo: "prisma/**"
---
# Database Instructions

- Schema changes require a migration: `pnpm db:migrate`
- New columns on existing tables MUST be nullable or have defaults
- Use `@map` and `@@map` for snake_case DB column names
- Always add indexes for foreign keys and frequently queried columns
- Never delete columns — deprecate with comments first
- Relations must have explicit `onDelete` behavior
```

### Test Files

```markdown
---
applyTo: "**/*.test.{ts,tsx}"
---
# Testing Instructions

- Use Vitest and React Testing Library
- Tests follow Arrange → Act → Assert pattern
- Use `describe` blocks for grouping, `it` for individual tests
- Mock external services; never hit real APIs in tests
- Test behavior, not implementation details
- Name tests: `it('should [expected behavior] when [condition]')`
- Prefer `getByRole` > `getByText` > `getByTestId` for queries
```

### Security-Sensitive Code

```markdown
---
applyTo: "src/auth/**"
---
# Auth Module Instructions

- Never log tokens, passwords, or PII
- Use bcrypt for password hashing (min 10 rounds)
- JWT tokens expire in 15 minutes; refresh tokens in 7 days
- Always validate JWT signature before trusting claims
- Rate limit all auth endpoints (5 attempts per minute)
- Return generic error messages ("Invalid credentials")
- Never reveal whether a username exists
```

### Configuration Files

```markdown
---
applyTo: "*.config.{ts,js,mjs}"
---
# Config File Instructions

- Do not modify config files unless explicitly asked
- These files are carefully tuned — explain any changes
- Always preserve existing comments in config files
- Test that the dev server starts after config changes
```

---

## The excludeAgent Field

Exclude specific Copilot features from receiving these instructions:

```yaml
---
applyTo: "**/*.ts"
excludeAgent: "code-review"
---
# These instructions are for Chat and Coding Agent only
# Code Review will NOT see them
```

| Value | Effect |
|-------|--------|
| `"code-review"` | Copilot Code Review won't receive these instructions |
| `"coding-agent"` | Copilot Coding Agent won't receive them |

**Use case**: Instructions that are relevant for writing code but not for reviewing it, or vice versa.

---

## Scoping Strategy

### Strategy 1: Domain-Based (Recommended)

```
.github/instructions/
├── frontend.instructions.md     # applyTo: "src/components/**"
├── api.instructions.md           # applyTo: "src/app/api/**"
├── database.instructions.md      # applyTo: "prisma/**"
├── testing.instructions.md       # applyTo: "**/*.test.*"
└── config.instructions.md        # applyTo: "*.config.*"
```

### Strategy 2: Language-Based

```
.github/instructions/
├── typescript.instructions.md   # applyTo: "**/*.{ts,tsx}"
├── python.instructions.md        # applyTo: "**/*.py"
├── sql.instructions.md           # applyTo: "**/*.sql"
└── markdown.instructions.md      # applyTo: "**/*.md"
```

### Strategy 3: Concern-Based

```
.github/instructions/
├── security.instructions.md     # applyTo: "src/auth/**"
├── performance.instructions.md   # applyTo: "src/workers/**"
├── accessibility.instructions.md # applyTo: "src/components/**"
└── data-integrity.instructions.md # applyTo: "src/db/**"
```

### Choosing a Strategy

| Strategy | Best for |
|----------|----------|
| Domain-based | Web apps with clear frontend/backend split |
| Language-based | Polyglot repos or monorepos |
| Concern-based | Enterprise apps with cross-cutting requirements |

Mix strategies as needed — use the one that makes each instruction file cohesive.

---

## Common Pitfalls

| Pitfall | Why It's Bad | Solution |
|---------|-------------|----------|
| Overlapping globs | Same file matches multiple instructions → context bloat | Make globs non-overlapping or keep rules complementary |
| Too many instruction files | 20+ files = token budget exceeded | Consolidate related rules into fewer files |
| applyTo too broad | `"**/*"` matches everything — same as repo-wide | Use copilot-instructions.md for repo-wide rules |
| Empty or near-empty files | 2 lines of instruction waste overhead | Merge small files into related ones |
| Contradicting repo-wide | Path says "use default exports" but repo says "named only" | Make exceptions explicit |

---

## Testing Path-Specific Instructions

### Verify Loading

1. Open a file that matches the `applyTo` pattern
2. Ask Copilot Chat something the instruction addresses
3. Check the References section — the `.instructions.md` file should appear

### Verify Non-Loading

1. Open a file that does NOT match the pattern
2. Ask the same question
3. The instruction file should NOT appear in References

### Verify Glob Correctness

Use your IDE or command line to test glob patterns:

```bash
# Find what files would match the applyTo pattern
find . -path "./src/components/**/*.tsx" -type f
```

```powershell
# PowerShell equivalent
Get-ChildItem -Path "src\components" -Recurse -Filter "*.tsx"
```
