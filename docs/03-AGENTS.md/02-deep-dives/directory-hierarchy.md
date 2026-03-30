# Directory Hierarchy & Resolution

> How nested AGENTS.md files work in monorepos, subprojects, and complex directory structures.

---

## The Core Rule

> **The nearest AGENTS.md in the directory tree takes precedence.**

When an AI agent is working on a file, it walks up the directory tree from the file's location to the repository root, looking for AGENTS.md files.

---

## Single Repository — No Hierarchy Needed

```
my-app/
├── AGENTS.md
├── src/
│   └── index.ts
└── tests/
    └── index.test.ts
```

All files use the root AGENTS.md. Simple.

---

## Monorepo — Per-Package Instructions

```
monorepo/
├── AGENTS.md                        ← Global defaults
├── apps/
│   ├── web/
│   │   ├── AGENTS.md                ← Web-specific (React, Next.js)
│   │   └── src/
│   │       └── page.tsx
│   └── mobile/
│       ├── AGENTS.md                ← Mobile-specific (React Native)
│       └── src/
│           └── App.tsx
├── packages/
│   ├── ui/
│   │   ├── AGENTS.md                ← UI library rules
│   │   └── src/
│   │       └── Button.tsx
│   └── shared/
│       └── src/
│           └── utils.ts             ← Falls through to root AGENTS.md
└── services/
    └── api/
        ├── AGENTS.md                ← API-specific (Express, Python, etc.)
        └── src/
            └── server.ts
```

### Resolution Examples

| File Being Edited | AGENTS.md Used |
|-------------------|----------------|
| `apps/web/src/page.tsx` | `apps/web/AGENTS.md` |
| `apps/mobile/src/App.tsx` | `apps/mobile/AGENTS.md` |
| `packages/ui/src/Button.tsx` | `packages/ui/AGENTS.md` |
| `packages/shared/src/utils.ts` | `AGENTS.md` (root — no closer match) |
| `services/api/src/server.ts` | `services/api/AGENTS.md` |

---

## Polyglot Monorepo — Language-Specific Instructions

```
platform/
├── AGENTS.md                        ← Common: git, CI, shared rules
├── frontend/                        ← TypeScript / React
│   ├── AGENTS.md
│   └── src/
├── backend/                         ← Go
│   ├── AGENTS.md
│   └── cmd/
├── ml/                              ← Python
│   ├── AGENTS.md
│   └── notebooks/
└── infra/                           ← Terraform / YAML
    ├── AGENTS.md
    └── modules/
```

### Root AGENTS.md (Shared Rules)

```markdown
# AGENTS.md (root)

## Git Workflow
- Branch naming: `feat/scope/description`
- Conventional commits required
- PRs target `main`

## CI
- All tests must pass before merge
- Security scan required

## Boundaries
- NEVER commit secrets
- NEVER modify `.github/workflows/` without reviewer approval
```

### frontend/AGENTS.md

```markdown
# AGENTS.md (frontend)

## Setup
- Package manager: pnpm
- Install: `pnpm install`
- Dev: `pnpm dev`
- Test: `pnpm test`

## Stack
TypeScript, React 18, Next.js 15, Tailwind CSS.

## Conventions
- Functional components only
- Named exports
- Server Components by default
```

### backend/AGENTS.md

```markdown
# AGENTS.md (backend)

## Setup
- Build: `go build ./...`
- Test: `go test ./... -v`
- Lint: `golangci-lint run`

## Stack
Go 1.22, Chi router, sqlc, PostgreSQL.

## Conventions
- Errors returned, not panicked
- Context passed as first argument
- Table-driven tests
```

---

## Inheritance vs. Override Behavior

### Tool-Dependent Behavior

Different tools handle multiple AGENTS.md files differently:

| Tool | Behavior |
|------|----------|
| **Codex CLI** | Reads nearest AGENTS.md; may merge with root |
| **GitHub Copilot Coding Agent** | Reads root AGENTS.md + task-relevant context |
| **Claude Code** | Merges all CLAUDE.md files from root to current |
| **Cursor** | Reads root .cursorrules; .cursor/rules/ for scoped rules |

### Safe Strategy: Design for Override

Since most tools read only the **nearest** AGENTS.md, include critical global rules in every file:

```markdown
# AGENTS.md (apps/web/)

## Global (from root)
- NEVER commit secrets
- Conventional commits required

## Web-Specific
- Stack: React, Next.js, TypeScript
- Test: `pnpm test`
```

**Trade-off**: Some duplication, but guaranteed each file is self-contained.

### Alternative: Keep Subdirectory Files Lean

If your tool merges root + nearest, keep subdirectory files minimal:

```markdown
# AGENTS.md (apps/web/)
# Extends root AGENTS.md with web-specific rules.

## Setup
- Dev: `pnpm dev`
- Test: `pnpm test`

## Conventions
- Server Components by default
```

---

## Patterns to Avoid

### ❌ AGENTS.md in Every Directory

```
src/
├── AGENTS.md
├── components/
│   ├── AGENTS.md        ← Too granular
│   ├── Button/
│   │   ├── AGENTS.md    ← WAY too granular
│   │   └── Button.tsx
```

**Problem**: Excessive hierarchy fragments context, increases maintenance burden, and most tools only read one level.

### ❌ Conflicting Instructions Across Levels

```markdown
# Root AGENTS.md
## Conventions
- Use default exports

# apps/web/AGENTS.md
## Conventions
- Use named exports only
```

**Problem**: If a tool merges both, the agent gets contradictory rules. Design for either override or additive, never both.

### ❌ Giant Root File + Empty Subdirectories

```
monorepo/
├── AGENTS.md              ← 500+ lines covering everything
├── apps/web/              ← No AGENTS.md (relies on bloated root)
└── apps/mobile/           ← No AGENTS.md (relies on bloated root)
```

**Problem**: Bloated root file wastes tokens on irrelevant context for each sub-project.

---

## Decision Framework

```
Do I need per-directory AGENTS.md?

Q: Do packages have different tech stacks?
  YES → Per-directory AGENTS.md
  NO  → Root-only AGENTS.md

Q: Do packages have different commands?
  YES → Per-directory AGENTS.md
  NO  → Root-only AGENTS.md

Q: Is my root AGENTS.md > 200 lines?
  YES → Split into per-directory files
  NO  → Root-only is fine

Q: Am I in a polyglot monorepo?
  YES → Per-directory AGENTS.md (almost always)
```
