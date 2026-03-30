# AGENTS.md for Copilot Users

> The cross-tool instruction standard supported by Copilot, Claude, Cursor, Codex, and Gemini.

---

## Overview

| Property | Value |
|----------|-------|
| **File** | `AGENTS.md` (any directory) |
| **Scope** | Nearest file in directory tree upward |
| **Auto-loaded** | Yes — by Copilot Coding Agent, Claude Code, Cursor Agent, Codex, Gemini CLI |
| **Priority** | Tool-dependent; typically repo-wide context |
| **Cross-tool** | ✅ The only file supported by all major AI tools |

---

## Why AGENTS.md Matters

If your team uses multiple AI tools (Copilot + Claude Code, or developers choose their own), `AGENTS.md` provides a **single source of truth** that every tool reads.

| Without AGENTS.md | With AGENTS.md |
|-------------------|----------------|
| `.github/copilot-instructions.md` for Copilot | ✅ Single file |
| `CLAUDE.md` for Claude Code | ✅ All tools read it |
| `.cursorrules` for Cursor | ✅ Consistency |
| Different conventions per tool | ✅ One standard |

---

## File Format

```markdown
# AGENTS.md

## Project
[Description and stack]

## Commands
[Build, test, lint commands]

## Conventions
[Coding standards]

## Directory Structure
[Where things live]

## Agent-Specific Notes
[Optional: instructions specific to AI agents]
```

### Directory Hierarchy

AGENTS.md uses a **nearest file wins** approach:

```
project/
├── AGENTS.md                    ← Root-level rules
├── src/
│   ├── AGENTS.md                ← Overrides for all of src/
│   ├── frontend/
│   │   └── AGENTS.md            ← Frontend-specific rules
│   └── backend/
│       └── AGENTS.md            ← Backend-specific rules
└── tests/
    └── AGENTS.md                ← Test-specific rules
```

When working on `src/frontend/Button.tsx`:
- Tool reads `src/frontend/AGENTS.md`
- Some tools also merge with parent `AGENTS.md` files (tool-dependent)

---

## When to Use AGENTS.md vs copilot-instructions.md

| Scenario | Use |
|----------|-----|
| Team uses only Copilot | `copilot-instructions.md` |
| Team uses multiple AI tools | `AGENTS.md` (primary) + tool-specific files for extras |
| Open source project | `AGENTS.md` (contributors use various tools) |
| Copilot-specific features needed | `copilot-instructions.md` (e.g., code review exclusions) |

### Dual-File Strategy

If you need both, keep AGENTS.md as the primary and copilot-instructions.md as a thin layer:

```markdown
# .github/copilot-instructions.md
See AGENTS.md for project conventions.

## Copilot-Specific
- Code review: focus on type safety and error handling
- Coding agent: always run `npm run test` after changes
```

---

## Practical Example

```markdown
# AGENTS.md

## Project
Task management API built with Express.js, TypeScript, Prisma, PostgreSQL.
REST API following OpenAPI 3.0 specification.

## Commands
- `npm install` — Install dependencies
- `npm run dev` — Start dev server (port 3001)
- `npm run test` — Run Jest tests
- `npm run test:watch` — Run tests in watch mode
- `npm run lint` — ESLint check
- `npm run build` — Compile TypeScript
- `npm run db:migrate` — Run Prisma migrations
- `npm run db:seed` — Seed database with test data

## Structure
src/
  routes/       # Express route handlers
  middleware/   # Auth, validation, error handling
  services/     # Business logic
  models/       # Prisma model extensions
  types/        # TypeScript type definitions
  utils/        # Shared utilities
prisma/         # Schema and migrations
tests/          # Test files (mirrors src/ structure)

## Conventions
- TypeScript strict mode; no `any`
- Express routes: router → middleware → service → response
- All input validated with Zod schemas
- Errors use custom AppError class with status codes
- Async handlers wrapped with `asyncHandler()` utility
- All responses follow `{ data: T }` or `{ error: { message, code } }` format

## Do Not
- Never commit .env files
- Never write raw SQL — use Prisma Client
- Never modify existing migrations — create new ones
- Never bypass validation middleware
- Never return stack traces in production responses

## Testing
- Unit tests for services and utilities
- Integration tests for API routes (using supertest)
- Test database is separate from development
- Run `npm run db:seed` before integration tests
```

---

## Tool Compatibility Notes

### Copilot Coding Agent
- Reads `AGENTS.md` from repo root
- Also reads `.github/copilot-instructions.md`
- Both are merged into context

### Claude Code
- Reads `CLAUDE.md` (native) and `AGENTS.md`
- Nearest file in directory tree wins
- Supports parent file inheritance

### Cursor
- Reads `.cursorrules` (native) and `AGENTS.md`
- Only reads from project root
- Does not support directory hierarchy

### OpenAI Codex
- Reads `AGENTS.md` natively
- Supports directory hierarchy

### Gemini CLI
- Reads `GEMINI.md` (native) and `AGENTS.md`
- Workspace-level configuration also supported

---

## Migration from Tool-Specific Files

If you're currently using `.cursorrules` or `CLAUDE.md`:

1. Create `AGENTS.md` in repo root
2. Copy content from your existing file
3. Remove tool-specific syntax (if any)
4. Test with your primary AI tool
5. Keep the old file as a thin redirect:

```markdown
# CLAUDE.md
See AGENTS.md for project conventions.
```
