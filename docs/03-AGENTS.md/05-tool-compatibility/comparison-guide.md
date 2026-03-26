# AGENTS.md vs CLAUDE.md vs .cursorrules

> A detailed side-by-side comparison of all major AI agent instruction formats.

---

## Quick Comparison

| Feature | AGENTS.md | CLAUDE.md | .cursorrules | copilot-instructions.md |
|---------|-----------|-----------|-------------|------------------------|
| **Format** | Markdown | Markdown | Plain text | Markdown |
| **Standard body** | AAIF / Linux Foundation | Anthropic (proprietary) | Cursor (proprietary) | GitHub (proprietary) |
| **Location** | Root + subdirectories | Root + subdirectories + ~/.claude/ | Root only (legacy) | .github/ |
| **Hierarchy** | ✅ Nearest wins | ✅ Merges all levels | ❌ Single file only | ✅ Path-specific rules |
| **Cross-tool** | ✅ Universal | ❌ Claude only | ❌ Cursor only | ❌ Copilot only |
| **Max size** | ~300 lines recommended | No hard limit (but ~1,500 recommended) | No hard limit | ~500-1,000 words |
| **Auto-loaded** | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes |
| **Tools supported** | 10+ | 1 (Claude) | 1 (Cursor) | 1 (Copilot) |

---

## Format Deep Dive

### AGENTS.md

```markdown
# AGENTS.md

## Setup
- Install: `pnpm install`
- Test: `pnpm test`

## Stack
TypeScript, React 18, Next.js 15, Prisma, PostgreSQL.

## Conventions
- TypeScript strict mode
- Named exports only
- Validate with Zod

## Boundaries
- NEVER commit secrets
- NEVER modify migrations/
```

**Strengths:**
- Universal — works with 10+ tools
- Simple — plain Markdown, no special syntax
- Standardized — governed by AAIF (Linux Foundation)
- Hierarchical — per-directory files for monorepos

**Weaknesses:**
- No tool-specific features (skills, memory, scoped rules)
- Behaviors vary slightly between tools
- No formal schema or validation

---

### CLAUDE.md

```markdown
# CLAUDE.md

## Project Overview
This is a Next.js 15 application with TypeScript.

## Commands
- `pnpm install` - Install dependencies
- `pnpm dev` - Start development server
- `pnpm test` - Run tests

## Code Style
- Use TypeScript strict mode
- Named exports only

## Memory
When the user says "save this", persist to project context.

## Skills
- `/test` - Run test suite and report results
- `/deploy-staging` - Deploy to staging environment
```

**Strengths:**
- Claude-specific features: skills, sub-agents, memory persistence
- Hierarchical merging: global (~/.claude/) → project → subdirectory
- Rich ecosystem within Claude Code
- User-level preferences separate from project rules

**Weaknesses:**
- Only works with Claude Code
- More complex to manage (multiple merge levels)
- Not recognized by other tools
- Proprietary format

---

### .cursorrules (Legacy) and .cursor/rules/*.mdc (Current)

#### Legacy: .cursorrules
```
You are a senior TypeScript developer.
Always use TypeScript strict mode.
Use named exports.
Never use any.
Follow the existing code patterns.
```

#### Current: .cursor/rules/*.mdc
```markdown
---
description: "React component conventions"
globs: ["src/components/**/*.tsx"]
alwaysApply: false
---

Use functional components with explicit Props interfaces.
Named exports only.
Co-locate tests in __tests__/ directories.
```

**Strengths:**
- .cursor/rules/ supports scoped rules per file pattern
- YAML frontmatter for metadata (globs, description, alwaysApply)
- Tight IDE integration in Cursor

**Weaknesses:**
- Only works in Cursor
- .cursorrules has no scoping — applied globally
- Different format from everything else
- No industry standardization

---

### .github/copilot-instructions.md

```markdown
# Copilot Instructions

## General
Always use TypeScript strict mode in this project.
Use named exports only.

## Path-Specific
Files matching `src/components/**` should follow React best practices
with functional components and hooks.
```

**Strengths:**
- Copilot-specific optimizations
- Path-specific `.instructions.md` files with `applyTo` globs
- Organization-level instructions (GitHub Enterprise)
- Tight integration with GitHub ecosystem

**Weaknesses:**
- Only works with GitHub Copilot
- Path-specific format not widely understood
- Code review instructions have separate 4,000 character limit

---

## When to Use Which

### Decision Matrix

| Scenario | Use |
|----------|-----|
| Open-source project (any contributors) | **AGENTS.md** only |
| Team uses only Copilot | **copilot-instructions.md** + **AGENTS.md** |
| Team uses only Claude | **CLAUDE.md** + **AGENTS.md** |
| Team uses only Cursor | **.cursor/rules/** + **AGENTS.md** |
| Multi-tool team | **AGENTS.md** (primary) + tool-specific (supplements) |
| Monorepo, multi-tool | **AGENTS.md** hierarchy (primary) + tool-specific per needs |

### The Universal Rule

> **Always have an AGENTS.md.** Add tool-specific files only for tool-specific features.

---

## Feature Availability by File

| Feature | AGENTS.md | CLAUDE.md | .cursor/rules/ | copilot-instructions.md |
|---------|:---------:|:---------:|:---------------:|:----------------------:|
| Build/test commands | ✅ | ✅ | ✅ | ✅ |
| Coding conventions | ✅ | ✅ | ✅ | ✅ |
| Boundaries (NEVER rules) | ✅ | ✅ | ✅ | ✅ |
| Per-directory hierarchy | ✅ | ✅ | ❌ | ❌ |
| Glob-based scoping | ❌ | ❌ | ✅ | ✅ |
| User-level preferences | ❌ | ✅ (~/.claude/) | ❌ | ✅ (personal settings) |
| Org-level standards | ❌ | ❌ | ❌ | ✅ (Enterprise) |
| Skills / sub-agents | ❌ | ✅ | ❌ | ❌ |
| Memory persistence | ❌ | ✅ | ❌ | ❌ |
| Cross-tool compatible | ✅ | ❌ | ❌ | ❌ |

---

## Migration Priority

If you're starting fresh:
1. Create AGENTS.md first (universal foundation)
2. Add tool-specific files only when you need features unique to that tool

If you have existing tool-specific files:
1. Keep them for tool-specific features
2. Add AGENTS.md with universal rules
3. Avoid duplicating rules between files
4. See [Migration Guide](migration-guide.md) for step-by-step instructions

---

*Next: [Dual-File Strategy](dual-file-strategy.md) →*
