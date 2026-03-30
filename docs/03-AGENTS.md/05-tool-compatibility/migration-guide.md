# Migration Guide

> Step-by-step instructions for migrating from .cursorrules, CLAUDE.md, or copilot-instructions.md to AGENTS.md.

---

## Why Migrate?

| Current State | Problem | After Migration |
|--------------|---------|-----------------|
| Only .cursorrules | Only Cursor developers benefit | All tools benefit |
| Only CLAUDE.md | Only Claude Code users benefit | All tools benefit |
| Only copilot-instructions.md | Only Copilot users benefit | All tools benefit |
| Multiple tool-specific files, no AGENTS.md | No universal baseline | Universal baseline + tool extensions |

---

## Migration 1: From .cursorrules to AGENTS.md

### Step 1: Read your current .cursorrules

```bash
cat .cursorrules
```

### Step 2: Create AGENTS.md

Map sections from .cursorrules to AGENTS.md:

| .cursorrules Content | AGENTS.md Section |
|---------------------|-------------------|
| Persona / role definition | Remove (not needed) |
| Tech stack description | ## Stack |
| Coding rules | ## Conventions |
| What to avoid | ## Boundaries |
| Testing instructions | ## Testing |
| Build/run commands | ## Setup |

### Step 3: Translate

**Before (.cursorrules):**
```
You are a senior TypeScript developer working on a Next.js 15 application.
Always use TypeScript strict mode.
Never use the `any` type.
Use functional components with hooks.
Tests are written with Vitest.
The project uses pnpm.
Don't modify anything in the legacy folder.
```

**After (AGENTS.md):**
```markdown
# AGENTS.md

## Setup
- Package manager: pnpm (NEVER use npm or yarn)
- Install: `pnpm install`
- Dev: `pnpm dev`
- Test: `pnpm test`
- Build: `pnpm build`

## Stack
TypeScript 5.4, React 18, Next.js 15 (App Router), Vitest, pnpm.

## Conventions
- TypeScript strict mode; never use `any`
- Functional components only
- Named exports (default exports only for Next.js pages)

## Boundaries
- NEVER modify files in legacy/
- NEVER commit .env files

## Testing
- Framework: Vitest
- Run: `pnpm test`
```

### Step 4: Handle Cursor-Specific Features

If your .cursorrules uses scoped rules, migrate those to .cursor/rules/*.mdc:

```yaml
# .cursor/rules/components.mdc
---
description: React component rules
globs: ["src/components/**/*.tsx"]
---

Use the component template pattern from src/templates/.
```

### Step 5: Decide on .cursorrules

Options:
- **Delete .cursorrules**: Cursor reads AGENTS.md
- **Keep both**: .cursorrules for Cursor-specific features, AGENTS.md for universal
- **Symlink**: `ln -s AGENTS.md .cursorrules` (if Cursor needs .cursorrules)

---

## Migration 2: From CLAUDE.md to AGENTS.md

### Step 1: Audit your CLAUDE.md

Identify which content is:
- **Universal** (any tool should follow these rules)
- **Claude-specific** (skills, memory, sub-agents)

### Step 2: Extract Universal Rules to AGENTS.md

**Before (CLAUDE.md only):**
```markdown
# CLAUDE.md

## Project
This is a FastAPI backend with PostgreSQL.

## Commands
- Install: `pip install -e .`
- Test: `pytest -xvs`
- Lint: `ruff check .`

## Rules
- Type hints on all functions
- Pydantic models for validation
- Never use raw SQL

## Skills
- /test - Run pytest and summarize
- /lint - Run ruff and auto-fix

## Memory
Remember the user's preferred file organization.
```

**After (AGENTS.md + CLAUDE.md):**

```markdown
# AGENTS.md (universal)

## Setup
- Install: `pip install -e .`
- Test: `pytest -xvs`
- Lint: `ruff check .`

## Stack
Python 3.12, FastAPI, Pydantic v2, PostgreSQL, SQLAlchemy 2.0.

## Conventions
- Type hints on all functions
- Pydantic models for API validation
- Never use raw SQL (use SQLAlchemy ORM)

## Boundaries
- NEVER commit .env or secrets
- NEVER modify alembic migration files after merge
```

```markdown
# CLAUDE.md (Claude-specific only)
# Universal rules are in AGENTS.md — DO NOT duplicate them here.

## Skills
- /test - Run `pytest -xvs` and summarize results
- /lint - Run `ruff check . --fix` and report changes

## Memory
Remember the user's preferred file organization patterns.
```

### Step 3: Test Both Files

1. In Claude Code: Verify both AGENTS.md and CLAUDE.md are loaded
2. In another tool: Verify AGENTS.md rules are followed

---

## Migration 3: From copilot-instructions.md to AGENTS.md

### Step 1: Audit .github/copilot-instructions.md

Identify:
- **Universal rules** → Move to AGENTS.md
- **Copilot-specific guidance** (code review, coding agent behavior) → Keep in copilot-instructions.md

### Step 2: Extract Universal Rules

**Before (.github/copilot-instructions.md only):**
```markdown
Always use TypeScript strict mode in this project.
Use pnpm as the package manager.
Follow the Airbnb style guide with modifications.
Use Vitest for testing.
Never use console.log — use the logger.
When reviewing PRs, check for missing error handling.
When assigned issues, keep changes minimal.
```

**After (AGENTS.md + copilot-instructions.md):**

```markdown
# AGENTS.md (universal)

## Setup
- Package manager: pnpm
- Test: `pnpm test`
- Lint: `pnpm lint`

## Conventions
- TypeScript strict mode
- Airbnb style guide (with project overrides in .eslintrc)
- Vitest for all tests

## Boundaries
- NEVER use console.log (use src/lib/logger)
```

```markdown
# .github/copilot-instructions.md (Copilot-specific)
# Universal rules are in AGENTS.md — DO NOT duplicate them here.

## Code Review
When reviewing PRs, specifically check for:
- Missing error handling in API routes
- console.log usage (should use logger)

## Coding Agent
When assigned issues:
- Keep changes minimal and focused
- Run `pnpm test` before opening PR
```

---

## Migration 4: Adding AGENTS.md to an Existing Multi-File Setup

If you already have multiple tool-specific files:

### Step 1: Identify Common Rules

```
.cursorrules:           "TypeScript strict mode"
CLAUDE.md:              "Use TypeScript strict, never any"
copilot-instructions:   "Always TypeScript strict mode"
```

All three say the same thing → AGENTS.md.

### Step 2: Create AGENTS.md with Shared Rules

Extract the intersection of all tool-specific files.

### Step 3: Remove Duplicates from Tool-Specific Files

Add the header comment to each:
```markdown
# Universal rules are in AGENTS.md — DO NOT duplicate them here.
```

### Step 4: Verify

Test each tool to confirm it reads both AGENTS.md and its specific file.

---

## Post-Migration Checklist

- [ ] AGENTS.md created with universal rules
- [ ] Tool-specific files contain only tool-specific content
- [ ] No rules duplicated between files
- [ ] Header comment in each tool-specific file pointing to AGENTS.md
- [ ] Tested with each tool the team uses
- [ ] CODEOWNERS updated to include AGENTS.md
- [ ] CI updated to validate AGENTS.md (optional)
- [ ] Team notified of the new file structure
