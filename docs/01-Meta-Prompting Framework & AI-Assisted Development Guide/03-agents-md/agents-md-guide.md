# AGENTS.md Guide

> The universal, cross-tool instruction file for AI coding agents.

---

## What It Is

`AGENTS.md` is an open standard for providing structured, machine-readable instructions to AI coding agents. Unlike tool-specific files (`copilot-instructions.md`, `CLAUDE.md`, `.cursorrules`), **AGENTS.md is designed to work across all major AI tools**.

**Supported by:**
- GitHub Copilot Coding Agent
- OpenAI Codex CLI
- Cursor (as fallback)
- Claude Code (via reference)
- Gemini CLI
- And growing...

**Governed by:** The Agentic AI Foundation (AAIF) under the Linux Foundation — ensuring vendor-neutral, open governance.

---

## Why Use AGENTS.md

| Scenario | Benefit |
|----------|---------|
| Team uses multiple AI tools | One file, all tools |
| Open source project | Contributors use whatever tool they prefer |
| Future-proofing | New tools will adopt the standard |
| Simpler maintenance | One instruction file to keep updated |

### AGENTS.md vs. copilot-instructions.md

You don't have to choose — **use both**:

- `copilot-instructions.md` → Copilot-specific features, Chat integration, Code Review
- `AGENTS.md` → Universal rules any AI agent should follow

When Copilot Coding Agent runs, it reads **both** files. Other tools read `AGENTS.md` and ignore `copilot-instructions.md`.

---

## File Location and Hierarchy

Place `AGENTS.md` at the repository root. For monorepos, you can place additional `AGENTS.md` files in subdirectories:

```
my-project/
├── AGENTS.md                    ← Root-level: project-wide rules
├── packages/
│   ├── frontend/
│   │   └── AGENTS.md            ← Frontend-specific overrides
│   └── backend/
│       └── AGENTS.md            ← Backend-specific overrides
```

**Resolution**: The nearest `AGENTS.md` in the directory tree takes precedence. If the AI is working on a file in `packages/frontend/`, it reads `packages/frontend/AGENTS.md` first.

---

## What to Include

### 1. Setup Commands (Put These First)

AI agents read top-down — critical commands should be early.

```markdown
## Setup
- Install: `npm install`
- Build: `npm run build`
- Test: `npm run test`
- Lint: `npm run lint`
- Dev server: `npm run dev`
```

### 2. Code Style and Conventions

```markdown
## Code Style
- TypeScript strict mode; no `any`
- Functional React components with named exports
- Tailwind CSS for styling; no CSS modules or inline styles
- Zod for runtime validation
- Single quotes, no semicolons (enforced by Prettier)
```

### 3. Explicit Dos and Don'ts

```markdown
## Do
- Run tests before committing
- Use the project's logger (`src/lib/logger.ts`)
- Follow existing patterns in the codebase
- Create co-located test files for all new modules

## Don't
- Don't add dependencies without approval
- Don't modify files in `vendor/` or `generated/`
- Don't commit secrets or API keys
- Don't disable linting or TypeScript rules
- Don't use `console.log` in production code
```

### 4. Project Architecture

```markdown
## Architecture
- Next.js App Router (pages in `src/app/`)
- API routes in `src/app/api/` (REST, validated with Zod)
- Shared types in `src/types/`
- Database: PostgreSQL via Prisma ORM
- Auth: NextAuth.js with JWT strategy
- State: React Server Components + client hooks where needed
```

### 5. Testing Requirements

```markdown
## Testing
- Framework: Jest + React Testing Library
- Run single test: `npm run test -- path/to/file.test.ts`
- All PRs must have passing tests
- Minimum coverage: 80% for new code
- E2E tests: `npm run test:e2e` (Playwright)
```

### 6. Safety and Compliance

```markdown
## Safety
- Never commit credentials; use `.env.local` for secrets
- Validate all user input server-side
- Sanitize HTML output to prevent XSS
- Use parameterized queries (Prisma handles this)
- Log security events to the audit trail
```

---

## Best Practices

### Writing for Machines

AGENTS.md is a "README for AI" — write differently than you would for humans:

| Human Documentation | AGENTS.md |
|---------------------|-----------|
| "See the wiki for setup" | `npm install && npm run dev` |
| "Follow best practices" | "Use named exports, no default exports" |
| "Be careful with the database" | "Never DROP columns; add nullable columns first" |
| Long paragraphs of context | Bullet points with commands |

### Structure Tips

- **Commands first**: Put executable commands at the top
- **Be explicit**: Ambiguity = agent guessing wrong
- **Use code blocks**: Real commands, not prose descriptions
- **Boundary-focused**: "always/never" is more useful than "try to"
- **Keep it under 2 pages**: Longer files dilute agent attention

### Monorepo Strategy

```
# Root AGENTS.md — shared rules
## Do
- Follow the monorepo commit convention: `type(scope): message`

# packages/frontend/AGENTS.md — frontend overrides
## Additional Rules
- Use Storybook for component development
- Follow the design system in `src/theme/`
```

---

## Relationship to Other Files

```
AGENTS.md          ← Universal rules (all AI tools read this)
   ↕ supplements
copilot-instructions.md   ← Copilot-specific features
   ↕ supplements
*.instructions.md          ← Path-scoped Copilot rules
```

**Recommendation for teams**: 
1. Put universal coding rules in `AGENTS.md`
2. Put Copilot-specific workflow and build details in `copilot-instructions.md`
3. Put domain-specific rules in `.instructions.md` files
