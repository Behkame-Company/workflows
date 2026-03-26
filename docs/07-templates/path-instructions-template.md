# Path-Specific Instructions Template

> Copy and customize for each domain in your codebase. Place in `.github/instructions/`.

---

## Frontend Components

```markdown
---
applyTo: "src/components/**/*.tsx,src/components/**/*.ts"
---

# Frontend Component Rules

## Component Pattern
- Use functional components with explicit TypeScript prop interfaces
- Co-locate tests: `ComponentName/index.tsx` + `ComponentName.test.tsx`
- Export from component barrel file when adding new public components

## Styling
- Use Tailwind CSS utility classes exclusively
- No inline styles, CSS modules, or styled-components
- Follow the 8px spacing grid from the design system

## Accessibility
- All interactive elements must have `aria-label` or associated `<label>`
- Use semantic HTML (`<button>`, `<nav>`, `<main>`, `<section>`)
- Ensure keyboard navigability for all interactive elements
- Color contrast must meet WCAG AA (4.5:1 for text)

## State Management
- Prefer React Server Components for data fetching
- Use client components (`'use client'`) only when interactivity is required
- Extract reusable hooks to `src/hooks/`
```

---

## API Routes

```markdown
---
applyTo: "src/app/api/**/*.ts"
---

# API Route Rules

## Validation
- Validate ALL input using Zod schemas from `src/lib/validators/`
- Return proper HTTP status codes (201 created, 400 bad request, 404 not found, 500 error)

## Error Handling
- Wrap handler logic in try/catch
- Log errors with `src/lib/logger.ts`, not console.error
- Return structured error responses: `{ error: string, details?: object }`
- Never expose internal error details to clients

## Patterns
- Use consistent CRUD naming: `GET`, `POST`, `PUT`, `DELETE` handlers
- Apply rate limiting middleware for public endpoints
- Document new endpoints in the OpenAPI spec
```

---

## Database / Prisma

```markdown
---
applyTo: "prisma/**/*"
---

# Database Rules

⚠️ CRITICAL: Database changes affect production data.

## Migrations
- Never drop columns without a multi-step deprecation plan
- New columns must be nullable OR have defaults
- Test migrations against a copy of production data before merging
- Name migrations descriptively: `npx prisma migrate dev --name add_user_roles`

## Schema Changes
- Run `npx prisma generate` after every schema modification
- Update seed data in `prisma/seed.ts` when adding new models
- Add appropriate indexes for columns used in WHERE clauses

## Queries
- Use Prisma Client for all database operations (no raw SQL unless justified)
- Always use `select` or `include` to limit returned fields
- Paginate large result sets
```

---

## Test Files

```markdown
---
applyTo: "**/*.test.*,**/*.spec.*,**/__tests__/**"
---

# Testing Rules

## Structure
- Use `describe` → `it` nesting pattern
- Name tests as sentences: `it('should display error when form is invalid')`
- One logical assertion per test when possible

## Patterns
- Test behavior, not implementation details
- Use `userEvent` over `fireEvent` for React components
- Mock external services; never make real API calls
- Use factories for test data (see `tests/factories/`)

## Coverage
- Each new module must have a co-located test file
- Cover happy path, error cases, and edge cases
- Don't test framework internals or third-party library behavior
```

---

## Security-Sensitive Code

```markdown
---
applyTo: "src/auth/**/*,src/middleware/**/*,src/lib/crypto/**/*"
---

# Security-Sensitive Code Rules

⚠️ This code handles authentication, authorization, and cryptography.

## Requirements
- All changes must be reviewed by a security-aware team member
- Never store plaintext passwords or tokens
- Use bcrypt for password hashing (cost factor 12+)
- JWT tokens must have expiration times
- Validate and sanitize all input before processing
- Log all authentication events to the audit trail

## Do Not
- Don't roll your own cryptography
- Don't disable CSRF protection
- Don't expose internal user IDs in public APIs
- Don't log sensitive data (passwords, tokens, PII)
```
