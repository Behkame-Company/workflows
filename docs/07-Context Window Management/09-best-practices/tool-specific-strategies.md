# Tool-Specific Strategies

> Context window management techniques specific to Copilot CLI, Claude Code, and Cursor.

---

## GitHub Copilot CLI

### Context Sources
- Open files in IDE
- `copilot-instructions.md` (repo-wide)
- `.instructions.md` (path-specific)
- Conversation history
- Referenced files via `#file`

### Strategies
1. **Close irrelevant tabs** — Copilot uses open files as context
2. **Use path-specific instructions** — `.instructions.md` in each directory
3. **Reference files explicitly** — `#file:src/auth/jwt.ts` instead of hoping it's in context
4. **New conversation per task** — Don't let unrelated history accumulate
5. **Keep instructions concise** — `copilot-instructions.md` should be <3K tokens

### Best Practices
```markdown
# copilot-instructions.md — keep it lean

## Project: E-Commerce Platform
Stack: Next.js 14, TypeScript, Prisma, PostgreSQL
Auth: JWT (src/auth/), Error format: { error, code, details }

## Standards
- Zod for input validation
- No `any` types
- Tests mirror source: src/foo.ts → tests/foo.test.ts
```

---

## Claude Code

### Context Sources
- `CLAUDE.md` files (auto-loaded)
- Tool results (file reads, searches, commands)
- Conversation history
- Memory tool (if enabled)

### Strategies
1. **CLAUDE.md as upfront context** — Loaded automatically, keep it focused
2. **Glob/grep before reading** — Discover, then load targeted
3. **Compaction is built in** — Claude Code handles this automatically
4. **Memory for cross-session** — Use memory files for persistent state
5. **Sub-agents for investigation** — Isolate exploration context

### CLAUDE.md Template
```markdown
# Project Overview
E-commerce platform (Next.js 14 + Express API)

# Quick Start
npm install && npm run dev

# Architecture
- /app — Next.js frontend (App Router)
- /api — Express backend
- /prisma — Database schema

# Conventions
- TypeScript strict, Zod validation on API boundaries
- Tests in __tests__/ mirroring source structure

# Current Focus
See .github/memory-bank/activeContext.md
```

---

## Cursor

### Context Sources
- Open files + `@file` references
- `.cursorrules` file
- `@codebase` for indexed search
- Conversation history

### Strategies
1. **`.cursorrules` for persistent context** — Like copilot-instructions.md
2. **`@codebase` for search** — AI-powered codebase search
3. **`@file` for targeted context** — Explicit file references
4. **Composer for multi-file work** — Separate context per task
5. **Tab for inline, Chat for complex** — Match tool to task

---

## Cross-Tool Common Practices

| Practice | Copilot | Claude Code | Cursor |
|----------|---------|-------------|--------|
| **Instruction file** | copilot-instructions.md | CLAUDE.md | .cursorrules |
| **Explicit file refs** | #file | Read tool | @file |
| **Codebase search** | N/A | grep/glob | @codebase |
| **New conversation** | ✅ Per task | ✅ Per task | ✅ Per task |
| **Close irrelevant files** | ✅ Critical | N/A | ✅ Helpful |
| **Compaction** | N/A | Built-in | N/A |
| **Memory** | Memory Bank files | Memory tool | .cursorrules |

---

## Key Takeaways

1. **Know your tool's context sources** — Different tools read different things
2. **Instruction files are universal** — Every tool has one; keep it lean
3. **Explicit > implicit** — Reference files explicitly, don't rely on auto-detection
4. **New conversations are free** — Start fresh for new tasks
5. **The principles are the same** — MVC, JIT retrieval, hygiene apply everywhere

---

## Next Steps

- 🔗 [Context Stuffing](../10-anti-patterns/context-stuffing.md) — The #1 anti-pattern
- 🔗 [Templates](../11-practical/templates.md) — Ready-to-use configurations
