# Instruction Files Overview

> A map of every instruction file type across AI coding tools — and when to use each.

---

## The Landscape

Modern AI coding tools support multiple instruction file types. Each serves a different purpose and scope:

```
Your Repository
├── .github/
│   ├── copilot-instructions.md          ← Copilot repo-wide rules
│   ├── instructions/
│   │   ├── react.instructions.md        ← Copilot path-specific rules
│   │   └── api.instructions.md          ← Copilot path-specific rules
│   └── prompts/
│       ├── add-feature.prompt.md        ← Copilot reusable prompts
│       └── code-review.prompt.md        ← Copilot reusable prompts
├── AGENTS.md                            ← Cross-tool agent instructions
├── CLAUDE.md                            ← Claude Code instructions
├── .cursorrules                         ← Cursor instructions (legacy)
├── .cursor/rules/*.mdc                  ← Cursor scoped rules (modern)
├── .clinerules                          ← Cline instructions
└── memory-bank/                         ← Persistent project memory
    ├── projectbrief.md
    ├── activeContext.md
    └── ...
```

---

## File Types at a Glance

### GitHub Copilot Files

| File | Location | Scope | Used By |
|------|----------|-------|---------|
| `copilot-instructions.md` | `.github/` | Entire repository | Copilot Chat, Copilot Agent |
| `*.instructions.md` | `.github/instructions/` | Files matching `applyTo` glob | Copilot Agent, Code Review |
| `*.prompt.md` | `.github/prompts/` | On-demand (user attaches) | Copilot Chat |
| `AGENTS.md` | Repo root (or any dir) | Nearest in directory tree | Copilot Agent + other tools |

### Other Tool Files

| File | Tool | Scope |
|------|------|-------|
| `CLAUDE.md` | Claude Code | Global → project → directory hierarchy |
| `.cursorrules` | Cursor (legacy) | Project root |
| `.cursor/rules/*.mdc` | Cursor (modern) | Scoped by file glob |
| `.clinerules` | Cline | Project root |
| `GEMINI.md` | Gemini CLI | Project root |

---

## Decision Guide: Which Files Do You Need?

### Minimum Viable Setup (Copilot-focused team)

```
.github/
└── copilot-instructions.md    ← Start here
```

Every repo should have this. It's the single most impactful file for AI-assisted development quality.

### Standard Team Setup

```
.github/
├── copilot-instructions.md           ← Repo-wide conventions
├── instructions/
│   ├── frontend.instructions.md      ← React/UI rules
│   ├── api.instructions.md           ← Backend rules
│   └── testing.instructions.md       ← Test patterns
└── prompts/
    └── add-feature.prompt.md         ← Standard feature workflow
AGENTS.md                             ← Cross-tool compatibility
```

### Full Framework Setup

```
.github/
├── copilot-instructions.md
├── instructions/
│   ├── frontend.instructions.md
│   ├── api.instructions.md
│   ├── database.instructions.md
│   └── testing.instructions.md
└── prompts/
    ├── add-feature.prompt.md
    ├── fix-bug.prompt.md
    └── code-review.prompt.md
AGENTS.md
memory-bank/
├── projectbrief.md
├── productContext.md
├── activeContext.md
├── systemPatterns.md
├── techContext.md
└── progress.md
```

---

## How Instructions Are Loaded

### Copilot Loading Order

1. **Organization custom instructions** (set by org admin on GitHub)
2. **Repository-wide** (`copilot-instructions.md`) — always loaded for repo context
3. **Path-specific** (`.instructions.md`) — loaded when working on matching files
4. **Prompt files** (`.prompt.md`) — loaded only when explicitly attached by user
5. **AGENTS.md** — loaded by Copilot Coding Agent (nearest in directory tree)

All applicable instructions merge. If conflicts exist, more specific files take precedence.

### Priority Resolution

```
User prompt (highest priority)
  ↑ Personal custom instructions
  ↑ Path-specific instructions (.instructions.md)
  ↑ Repository instructions (copilot-instructions.md)
  ↑ Organization instructions (lowest priority)
```

---

## Choosing Between Instruction Types

| You want to... | Use |
|-----------------|-----|
| Set coding conventions for the whole repo | `copilot-instructions.md` |
| Apply different rules to frontend vs backend | `*.instructions.md` with `applyTo` |
| Create a repeatable workflow (e.g., "add a feature") | `*.prompt.md` |
| Support team members using different AI tools | `AGENTS.md` |
| Maintain project knowledge across sessions | `memory-bank/` |
| Define agent-specific behavior for Copilot Coding Agent | `AGENTS.md` or `copilot-instructions.md` |

---

## Best Practices

### Do

- ✅ Start with `copilot-instructions.md` — it gives the highest ROI
- ✅ Keep instructions **actionable and specific** (not "write clean code")
- ✅ Include build/test/lint commands so the AI can self-validate
- ✅ Use path-specific files for domain boundaries (frontend, backend, database)
- ✅ Version control all instruction files — they're part of your codebase
- ✅ Review and update instructions after major refactors

### Don't

- ❌ Duplicate content across instruction files — keep it DRY
- ❌ Write instructions longer than 2 pages per file (context budget)
- ❌ Include task-specific details in repo-wide instructions
- ❌ Forget to test that instructions actually change AI behavior
- ❌ Let instruction files go stale — schedule quarterly reviews
