# Custom Instructions for Copilot Agents

> Configure agent behavior through layered instruction files — repo-wide rules, path-specific guidance, and reusable prompt files that shape how Copilot agents work in your project.

---

## Three Levels of Instructions

Copilot uses a layered instruction system. All instruction files now **combine** — they don't override each other in a priority-based fallback.

```
┌───────────────────────────────────────────────────────────────┐
│                Instruction File Hierarchy                      │
├───────────────────────────────────────────────────────────────┤
│                                                               │
│  Level 1: copilot-instructions.md                             │
│  ─────────────────────────────────                            │
│  Location: .github/copilot-instructions.md                    │
│  Scope:    Entire repository                                  │
│  Purpose:  Global coding standards, architecture rules        │
│                                                               │
│  Level 2: .instructions.md                                    │
│  ────────────────────────                                     │
│  Location: Any directory (e.g., tests/.instructions.md)       │
│  Scope:    Files within that directory tree                   │
│  Purpose:  Context-specific rules for a subsystem             │
│                                                               │
│  Level 3: .prompt.md                                          │
│  ────────────────────                                         │
│  Location: Anywhere in the project                            │
│  Scope:    Explicitly referenced or auto-discovered           │
│  Purpose:  Reusable prompt templates with tool access         │
│                                                               │
│  All levels COMBINE ── agent sees all applicable rules        │
│                                                               │
└───────────────────────────────────────────────────────────────┘
```

## Level 1: copilot-instructions.md

The repo-wide instruction file. Every Copilot agent (coding agent, CLI, IDE) reads this.

```markdown
<!-- .github/copilot-instructions.md -->

## Project Overview
This is a Next.js 14 e-commerce application using TypeScript,
Prisma ORM, and PostgreSQL.

## Code Style
- Use TypeScript strict mode — no `any` types
- Prefer functional components with React hooks
- Use named exports exclusively (no default exports)
- Keep files under 300 lines — extract when larger

## Testing
- Test command: `npm test`
- Framework: Jest + React Testing Library
- Test files: `__tests__/ComponentName.test.tsx` (co-located)
- Minimum coverage: 80% for new code
- Always test error states and edge cases

## Architecture
- API routes: `src/app/api/`
- Components: `src/components/` (atomic design)
- Services: `src/services/` (business logic, no UI imports)
- Types: `src/types/` (shared TypeScript interfaces)
- Database: `prisma/schema.prisma`

## Forbidden Patterns
- Never use `console.log` in production code (use logger service)
- Never commit `.env` files
- Never use inline styles (use Tailwind classes)
- Never mutate function parameters
```

## Level 2: .instructions.md (Path-Specific)

Place `.instructions.md` in any directory to provide context-specific rules. The agent loads these when working on files in that directory.

### Example: Test Directory Instructions

```markdown
<!-- tests/.instructions.md -->

## Test Conventions for This Project

### Structure
- Use `describe` blocks grouped by function/component
- Use `it` with behavior-driven descriptions: "it should return 404 for missing user"
- One assertion per test when practical

### Playwright E2E Tests
- Page objects live in `tests/pages/`
- Test data factories in `tests/fixtures/`
- Always clean up test data in `afterEach`
- Use `data-testid` attributes for selectors, never CSS classes

### Running Tests
- Unit: `npm test`
- E2E: `npx playwright test`
- Single file: `npx playwright test tests/auth.spec.ts`

### Mocking
- Mock external APIs with MSW (already configured in tests/setup.ts)
- Never mock internal services in integration tests
- Database tests use the test database (see .env.test)
```

### Example: API Directory Instructions

```markdown
<!-- src/api/.instructions.md -->

## API Route Conventions

### Structure
Each route file exports named handlers: GET, POST, PUT, DELETE

### Validation
- Use Zod schemas for request validation
- Schemas live in the same file as the route
- Return 400 with structured error for validation failures

### Error Handling
- All routes wrapped in try/catch
- Use AppError class from src/lib/errors.ts
- Never expose internal errors to clients

### Response Format
Always use: { data: T } for success, { error: { message, code } } for errors
```

## Level 3: .prompt.md (Reusable Prompts)

Prompt files are reusable templates that can include tool configurations. They're useful for codifying common agent tasks.

```markdown
<!-- .github/prompts/add-api-endpoint.prompt.md -->

## Task: Add a New API Endpoint

When creating a new API endpoint, follow these steps:

1. Create the route file in `src/app/api/[resource]/route.ts`
2. Define Zod validation schemas for request/response
3. Implement the handler using the service layer
4. Add the route to `src/types/api.ts` type definitions
5. Create tests in `tests/api/[resource].test.ts`
6. Update the API documentation in `docs/api/`

### Template

```typescript
import { NextRequest, NextResponse } from "next/server";
import { z } from "zod";

const RequestSchema = z.object({
  // Define request shape
});

export async function POST(request: NextRequest) {
  const body = await request.json();
  const validated = RequestSchema.parse(body);
  // Implementation using service layer
  return NextResponse.json({ data: result });
}
```
```

## Environment Setup for the Coding Agent

The Copilot Coding Agent (cloud-based) also needs `copilot-setup-steps.yml` to configure its container:

```yaml
# .github/copilot-setup-steps.yml

steps:
  - name: Install Node.js dependencies
    run: npm ci

  - name: Set up database
    run: |
      npx prisma generate
      npx prisma db push

  - name: Verify environment
    run: |
      npm run build
      npm test -- --passWithNoTests
```

This file is **only** for the cloud coding agent. The CLI agent uses your local environment directly.

## Key Instruction Patterns for Agents

### Pattern 1: Define Test Commands Explicitly

```markdown
## Testing
Run all tests: `npm test`
Run single test: `npm test -- --testPathPattern=<pattern>`
Run with coverage: `npm test -- --coverage`

Always run tests after making changes.
If tests fail, fix the code — never modify tests to make them pass
unless the test itself is incorrect.
```

### Pattern 2: Specify Code Style with Examples

```markdown
## Naming Conventions
- Components: PascalCase (`UserProfile.tsx`)
- Hooks: camelCase with `use` prefix (`useAuth.ts`)
- Utils: camelCase (`formatDate.ts`)
- Constants: SCREAMING_SNAKE_CASE (`MAX_RETRY_COUNT`)
- Types: PascalCase with descriptive suffixes (`UserResponse`, `CreateUserInput`)
```

### Pattern 3: Provide Architecture Context

```markdown
## Request Flow
1. Client sends request to Next.js API route
2. Route validates input with Zod
3. Route calls service function
4. Service queries database via Prisma
5. Service returns typed result
6. Route formats and returns response

Never skip layers — routes must not query the database directly.
```

### Pattern 4: List Forbidden Patterns

```markdown
## Do Not
- Import from `src/services/` in component files (use hooks instead)
- Use `any` type (use `unknown` and narrow)
- Write SQL queries (use Prisma client)
- Add dependencies without team discussion (comment in PR)
- Modify migration files after they've been committed
```

## Complete Instruction File Template

```markdown
<!-- .github/copilot-instructions.md — Full Template -->

## Project: [Name]
[One-sentence description]

## Tech Stack
- Language: [e.g., TypeScript 5.x]
- Framework: [e.g., Next.js 14]
- Database: [e.g., PostgreSQL via Prisma]
- Testing: [e.g., Jest + Playwright]
- Styling: [e.g., Tailwind CSS]

## Commands
- Install: `npm ci`
- Dev: `npm run dev`
- Build: `npm run build`
- Test: `npm test`
- Lint: `npm run lint`
- Type check: `npx tsc --noEmit`

## Code Conventions
[3-5 most important rules]

## Architecture
[Key directories and their purposes]

## Testing Requirements
[Test patterns, coverage requirements, what to test]

## Common Tasks
- Adding a new API endpoint: [steps or link to prompt file]
- Adding a new component: [steps or link to prompt file]
- Database migrations: [steps]

## Forbidden Patterns
[Things the agent must never do]
```

## How Instructions Combine

When the agent works on a file like `tests/api/users.test.ts`, it sees:

```
Combined Instructions:
├── .github/copilot-instructions.md     (repo-wide rules)
├── tests/.instructions.md              (test-specific rules)
├── tests/api/.instructions.md          (API test rules, if exists)
└── Any referenced .prompt.md files
```

All instructions are merged and sent to the agent together. Write them to complement each other, not conflict.

## Tips for Effective Instructions

1. **Keep it concise** — agents follow shorter, focused docs better than long ones
2. **Use positive instructions** — "Use async/await" works better than "Don't use callbacks"
3. **Specify exact commands** — `npm test -- --coverage` beats "run tests with coverage"
4. **Include examples** — a code snippet is worth a hundred words of description
5. **Update regularly** — instructions should evolve with your codebase
