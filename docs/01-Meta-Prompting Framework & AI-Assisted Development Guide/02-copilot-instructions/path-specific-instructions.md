# Path-Specific Instructions

> Scoped rules that apply only when Copilot works on matching files.

---

## What They Are

Path-specific instructions are `.instructions.md` files stored in `.github/instructions/`. Each file uses a `applyTo` frontmatter field with glob patterns to define which files the instructions apply to.

They are **additive** — when a file matches, these instructions are merged with the repository-wide `copilot-instructions.md`.

**Supported by:**
- Copilot Coding Agent
- Copilot Code Review

---

## When to Use

| Scenario | Example |
|----------|---------|
| Frontend and backend have different conventions | Separate React rules from API rules |
| Test files need specific patterns | Testing framework conventions |
| Database files need extra caution | Migration safety rules |
| Different languages in a monorepo | Python rules for scripts, TypeScript for app |
| Security-sensitive paths | Extra scrutiny for auth/payment code |

---

## File Structure

```
.github/
└── instructions/
    ├── frontend.instructions.md       # React component rules
    ├── api.instructions.md            # Backend API rules
    ├── database.instructions.md       # Prisma/migration rules
    ├── testing.instructions.md        # Test file conventions
    └── security/
        └── auth.instructions.md       # Auth module rules
```

You can organize with subdirectories — only the file extension `.instructions.md` matters.

---

## Anatomy of an Instructions File

```markdown
---
applyTo: "src/components/**/*.tsx,src/components/**/*.ts"
---

# React Component Instructions

## Conventions
- Use functional components with TypeScript interfaces for props
- Every component must have a co-located test file (`ComponentName.test.tsx`)
- Use Tailwind CSS utility classes; no inline styles or CSS modules
- Extract hooks into `src/hooks/` when reused across 2+ components

## Component Structure
```tsx
interface Props {
  // Always define explicit prop interfaces
}

export function ComponentName({ prop1, prop2 }: Props) {
  // Component logic
}
```

## Accessibility
- All interactive elements must have `aria-label` or associated `<label>`
- Use semantic HTML elements (`<button>`, `<nav>`, `<main>`)
- Test with keyboard navigation
```

---

## Glob Pattern Reference

| Pattern | Matches |
|---------|---------|
| `**/*.ts` | All TypeScript files recursively |
| `**/*.tsx,**/*.ts` | Multiple patterns (comma-separated) |
| `src/components/**/*.tsx` | TSX files under components |
| `src/app/api/**/*` | All API route files |
| `prisma/**/*` | All Prisma-related files |
| `**/*.test.*,**/*.spec.*` | All test files |
| `*` | All files in current directory only |
| `**` or `**/*` | All files in all directories |

---

## Excluding from Specific Agents

Use `excludeAgent` to limit which AI agent reads the file:

```yaml
---
applyTo: "**/*.tsx"
excludeAgent: "code-review"
---
# These instructions are only used by Copilot Coding Agent, not Code Review
```

Valid values:
- `"code-review"` — exclude from Copilot Code Review
- `"coding-agent"` — exclude from Copilot Coding Agent

---

## Practical Examples

### API Routes

```markdown
---
applyTo: "src/app/api/**/*.ts"
---

# API Route Instructions

- All routes must validate input using Zod schemas from `src/lib/validators/`
- Return proper HTTP status codes (201 for creation, 404 for not found, etc.)
- Always wrap handler logic in try/catch with structured error responses
- Log errors using `src/lib/logger.ts`, not console.error
- Include rate limiting middleware for public endpoints
- Document new endpoints in the OpenAPI spec at `docs/api/openapi.yaml`
```

### Database Migrations

```markdown
---
applyTo: "prisma/**/*"
---

# Database Instructions

⚠️ CRITICAL: Database changes affect production data.

- Never drop columns without a multi-step migration plan
- Always add new columns as optional (nullable) first
- Test migrations against a copy of production data before merging
- Run `npx prisma migrate dev --name descriptive_name` to create migrations
- Run `npx prisma generate` after schema changes
- Update seed data in `prisma/seed.ts` when adding new models
```

### Test Files

```markdown
---
applyTo: "**/*.test.*,**/*.spec.*,**/__tests__/**"
---

# Testing Instructions

- Use `describe` → `it` nesting pattern
- Test behavior, not implementation details
- Use `userEvent` over `fireEvent` for React component tests
- Mock external services; never make real API calls in tests
- Each test file should be co-located with the file it tests
- Aim for one assertion per test when possible
- Name tests as sentences: `it('should display error when form is invalid')`
```

---

## Interaction with Repository-Wide Instructions

When both apply, they are **merged**:

```
copilot-instructions.md says:     "Use TypeScript strict mode"
frontend.instructions.md says:    "Use Tailwind for all styling"

→ Copilot receives BOTH instructions when working on a .tsx file
```

⚠️ **Avoid contradictions** between repo-wide and path-specific files. If `copilot-instructions.md` says "use CSS modules" but `frontend.instructions.md` says "use Tailwind," the AI will be confused.

---

## Size Guidelines

- Keep each file focused on **one domain** (frontend, backend, database, etc.)
- Target **half a page to one page** per file
- If a file grows beyond a page, split into sub-domains
