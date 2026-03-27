# Hybrid Retrieval Strategies

> Combining upfront context with on-demand retrieval for the best of both worlds.

---

## The Hybrid Model

Neither pure eager loading nor pure just-in-time retrieval is optimal. The hybrid approach loads a small, high-signal context upfront and retrieves details on demand.

```
┌─────────────────────────────────────────┐
│  UPFRONT (loaded at session start)       │
│                                          │
│  • System prompt (rules, persona)        │
│  • CLAUDE.md / copilot-instructions.md   │
│  • Progress notes (what's been done)     │
│  • Project structure overview            │
│  Total: 3K-8K tokens                     │
├─────────────────────────────────────────┤
│  ON-DEMAND (loaded when needed)          │
│                                          │
│  • Specific source files                 │
│  • Search results                        │
│  • Test output                           │
│  • API documentation                     │
│  • Error logs                            │
│  Total: Varies per task                  │
└─────────────────────────────────────────┘
```

---

## What Goes Upfront vs. On-Demand

| Upfront (Always in Context) | On-Demand (Retrieved as Needed) |
|------------------------------|-------------------------------|
| Project tech stack | Specific source files |
| Architecture overview | Detailed API docs |
| Active task description | Test results |
| Key constraints | Error traces |
| File structure map | Git diffs |
| Coding standards | Dependency documentation |

### Decision Rule
> If the agent needs this info for **every** task → upfront.
> If the agent needs this info for **this specific** task → on-demand.

---

## Example: copilot-instructions.md as Upfront Context

```markdown
# Project: E-Commerce Platform

## Architecture
- Frontend: Next.js 14 (App Router) in /app
- Backend: Express API in /api
- Database: PostgreSQL with Prisma ORM
- Auth: JWT (src/auth/)

## Key Conventions
- All API routes validate input with Zod
- Error responses use standard format: { error, code, details }
- Tests mirror source structure: src/foo.ts → tests/foo.test.ts

## Current State
See .github/memory-bank/activeContext.md for current work.

## When You Need More Context
- Read specific files with targeted line ranges
- Use grep to find patterns across the codebase
- Check git log for recent changes
```

This gives the agent a **map** — the actual **territory** is explored on demand.

---

## Key Takeaways

1. **Map upfront, territory on demand** — Overview always, details when needed
2. **Upfront: 3K-8K tokens max** — Enough to orient, not enough to clutter
3. **On-demand: targeted retrieval** — Specific files, specific lines
4. **The decision rule** — Needed for every task? Upfront. Specific task? On-demand.
5. **As models improve, more autonomy** — Smarter models need less upfront

---

## Next Steps

- 🔗 [Agentic Search](agentic-search.md) — AI discovers what it needs
- 🔗 [Progressive Disclosure](progressive-disclosure.md) — Layer-by-layer building
