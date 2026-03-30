# Dual-File Strategy

> Using AGENTS.md alongside tool-specific instruction files without duplication or conflict.

---

## The Problem

You want:
1. **Universal rules** that work across all tools (AGENTS.md)
2. **Tool-specific features** that only one tool supports (CLAUDE.md, .cursorrules, etc.)

But you don't want:
- Duplicate rules that drift out of sync
- Conflicting instructions between files
- Maintenance burden of multiple files

---

## The Dual-File Architecture

```
┌─────────────────────────────────────────────┐
│                AGENTS.md                      │
│          (Universal Foundation)                │
│                                               │
│  ● Setup commands                             │
│  ● Stack & versions                           │
│  ● Conventions                                │
│  ● Boundaries                                 │
│  ● Testing                                    │
│  ● Git workflow                               │
└─────────────────┬───────────────────────────┘
                  │
        ┌─────────┼──────────┐
        ▼         ▼          ▼
  ┌──────────┐ ┌──────────┐ ┌────────────────────┐
  │CLAUDE.md │ │.cursor/  │ │copilot-instructions│
  │          │ │rules/    │ │.md                  │
  │ Claude-  │ │          │ │                     │
  │ specific │ │ Cursor-  │ │ Copilot-specific    │
  │ features │ │ specific │ │ features            │
  │ only     │ │ features │ │ only                │
  └──────────┘ └──────────┘ └────────────────────┘
```

### Rule: Universal → AGENTS.md. Tool-specific → Tool file.

---

## What Goes Where

### In AGENTS.md (Universal)

```markdown
# AGENTS.md

## Setup
- Install: `pnpm install`
- Test: `pnpm test`
- Build: `pnpm build`

## Stack
TypeScript 5.4, React 18, Next.js 15, Prisma, PostgreSQL.

## Conventions
- TypeScript strict mode; never use `any`
- Named exports only
- Validate API input with Zod

## Boundaries
- NEVER commit secrets
- NEVER modify migrations/
- NEVER disable lint rules

## Testing
- Framework: Vitest + React Testing Library
- All new logic must have tests
```

### In .github/copilot-instructions.md (Copilot-Specific)

```markdown
# Copilot-specific instructions
# Universal rules are in AGENTS.md — DO NOT duplicate them here.

## Code Review
When reviewing PRs, check for:
- Missing Zod validation on API inputs
- console.log usage (should use logger)
- Default exports (should be named exports)

## Coding Agent
When assigned issues labeled `good-first-issue`:
- Keep changes minimal and focused
- Add comprehensive inline comments for learning
```

### In CLAUDE.md (Claude-Specific)

```markdown
# Claude-specific instructions
# Universal rules are in AGENTS.md — DO NOT duplicate them here.

## Skills
- `/test` — Run `pnpm test` and summarize results
- `/lint` — Run `pnpm lint` and fix auto-fixable issues
- `/deploy-staging` — Deploy to staging via `pnpm deploy:staging`

## Memory
- Remember user preferences across sessions
- Track which files were recently modified
```

### In .cursor/rules/components.mdc (Cursor-Specific)

```yaml
---
description: React component conventions
globs: ["src/components/**/*.tsx"]
alwaysApply: false
---
```

```markdown
# Cursor-specific component rules
# Universal rules are in AGENTS.md — DO NOT duplicate them here.

When editing React components:
- Use the component template from src/templates/component.tsx
- Run `pnpm test -- --filter=components` after changes
```

---

## Preventing Duplication

### The Comment Convention

In every tool-specific file, add a header comment:

```markdown
# [Tool] specific instructions
# Universal rules are in AGENTS.md — DO NOT duplicate them here.
```

This tells both humans and future AI agents that AGENTS.md is the source of truth.

### The Audit Rule

During quarterly reviews:
1. Read AGENTS.md
2. Read each tool-specific file
3. Check: Is anything in a tool-specific file that should be in AGENTS.md?
4. Check: Is anything duplicated between files?
5. Move duplicates to AGENTS.md; remove from tool-specific files

---

## Conflict Resolution

When AGENTS.md and a tool-specific file conflict, the tool typically uses its own file.

### Example Conflict

```markdown
# AGENTS.md
## Conventions
- Named exports only

# .cursor/rules/pages.mdc
Use default exports for page components.
```

**In Cursor**: The .cursor/rules/ file wins for matched files.
**In Copilot**: AGENTS.md applies (no .cursor/ rules).

### How to Avoid Conflicts

1. **AGENTS.md**: General rules that apply everywhere
2. **Tool-specific**: Only exceptions or tool-unique features
3. **Document exceptions**: Note in AGENTS.md when a tool-specific file overrides

```markdown
# AGENTS.md
## Conventions
- Named exports only
- Exception: Next.js pages/layouts use default exports (enforced by framework)
```

---

## Decision Framework

For every instruction you want to add, ask:

```
Q1: Does this rule apply regardless of which AI tool is used?
    YES → AGENTS.md
    NO  → Tool-specific file

Q2: Does this rule use a tool-specific feature? (skills, scoping, memory)
    YES → Tool-specific file
    NO  → AGENTS.md

Q3: Is this a universal coding convention?
    YES → AGENTS.md
    NO  → Probably tool-specific

Q4: Would a contributor using a different tool need this?
    YES → AGENTS.md
    NO  → Tool-specific file
```

---

## File Tree Example (Complete Setup)

```
project/
├── AGENTS.md                           ← Universal (all tools read this)
├── CLAUDE.md                           ← Claude Code (skills, memory)
├── .cursorrules                        ← Cursor legacy (if needed)
├── .cursor/
│   └── rules/
│       ├── components.mdc              ← Cursor scoped rules
│       └── api-routes.mdc
├── .github/
│   ├── copilot-instructions.md         ← Copilot repo-wide
│   └── instructions/
│       ├── frontend.instructions.md    ← Copilot path-specific
│       └── api.instructions.md
└── src/
    └── ...
```

**Maintenance**: 1 universal file + 1-2 tool-specific files. Total effort: ~20 minutes per quarter.
