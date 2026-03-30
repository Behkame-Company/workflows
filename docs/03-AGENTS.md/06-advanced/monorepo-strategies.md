# Monorepo Strategies for AGENTS.md

> Patterns for polyglot, multi-package repositories with different tech stacks, teams, and conventions.

---

## Why Monorepos Need Special Treatment

A monorepo contains multiple projects with different:
- **Languages** (TypeScript, Python, Go, Rust)
- **Frameworks** (React, FastAPI, Chi)
- **Build tools** (pnpm, pip, go build)
- **Test commands** (vitest, pytest, go test)
- **Conventions** (named exports vs. error returns vs. trait implementations)

A single root AGENTS.md can't effectively serve all of these.

---

## Strategy 1: Root + Package Files (Recommended)

```
monorepo/
├── AGENTS.md                    ← Shared: git, CI, security, boundaries
├── apps/
│   ├── web/
│   │   ├── AGENTS.md            ← TypeScript, React, Next.js
│   │   └── src/
│   ├── mobile/
│   │   ├── AGENTS.md            ← React Native, Expo
│   │   └── src/
│   └── admin/
│       ├── AGENTS.md            ← TypeScript, Vue 3
│       └── src/
├── packages/
│   ├── ui/
│   │   ├── AGENTS.md            ← Component library, Storybook
│   │   └── src/
│   ├── api-client/
│   │   └── src/                 ← Falls through to root AGENTS.md
│   └── shared/
│       └── src/                 ← Falls through to root AGENTS.md
└── services/
    ├── api/
    │   ├── AGENTS.md            ← Python, FastAPI
    │   └── src/
    └── worker/
        ├── AGENTS.md            ← Go, queue processing
        └── cmd/
```

### Root AGENTS.md (40-60 lines)

```markdown
# AGENTS.md (root)

## Monorepo Overview
This is a Turborepo monorepo. Each package has its own AGENTS.md.
For package-specific rules, see the AGENTS.md in that package.

## Package Manager
pnpm workspaces. NEVER use npm or yarn.

## Git Workflow
- Branch: `feat/scope/description` (scope = package name)
- Commits: conventional commits with scope: `feat(web): add login page`
- PRs target `main`
- Changes to shared/ require extra review

## CI
- All tests must pass before merge
- Affected packages detected automatically by Turborepo

## Boundaries (Global)
- NEVER commit .env files or secrets
- NEVER modify .github/workflows/ without DevOps review
- NEVER add root-level dependencies (add to specific packages)
- NEVER import between apps/ (use packages/ for shared code)
```

### Package-Specific AGENTS.md (30-60 lines each)

```markdown
# AGENTS.md (apps/web)

## Setup
- Install (from root): `pnpm install`
- Dev: `pnpm --filter web dev`
- Test: `pnpm --filter web test`
- Build: `pnpm --filter web build`
- Lint: `pnpm --filter web lint`

## Stack
TypeScript 5.4, React 18, Next.js 15 (App Router), Tailwind CSS 3.4.

## Conventions
- Server Components by default
- 'use client' only for interactive components
- Named exports only
- Validate forms with Zod + react-hook-form

## Boundaries
- NEVER import from apps/mobile or apps/admin
- NEVER import from services/
- Shared code goes in packages/
```

---

## Strategy 2: Root-Only with Section Tags

For smaller monorepos (2-3 packages with similar tech):

```markdown
# AGENTS.md

## Setup
- Install: `pnpm install`
- Test all: `pnpm test`
- Test web: `pnpm --filter web test`
- Test api: `pnpm --filter api test`

## Stack
TypeScript 5.4 throughout. React 18 + Next.js 15 (web), Express 5 (api).

## Conventions (All Packages)
- TypeScript strict mode
- Named exports only
- Zod for validation

## Conventions (Web Only)
- Server Components by default
- Tailwind CSS for styling

## Conventions (API Only)
- Express middleware pattern
- Prisma for database access

## Boundaries
- NEVER import between apps
- Shared code goes in packages/
```

**When to use**: Small monorepo (< 5 packages) with similar tech stacks.
**When to avoid**: Polyglot monorepo or many packages (file becomes too long).

---

## Strategy 3: Polyglot with Language Boundaries

For monorepos with multiple programming languages:

```markdown
# AGENTS.md (root — 30 lines max)

## Languages
- frontend/: TypeScript
- backend/: Python
- infra/: Terraform + YAML
- ml/: Python + Jupyter

Each directory has its own AGENTS.md with language-specific rules.

## Global Rules
- NEVER commit secrets
- Conventional commits with scope
- All tests must pass before merge
```

Each language directory gets a full AGENTS.md optimized for that stack.

---

## Pattern: Shared Package Rules

Packages in `packages/` often don't need their own AGENTS.md if they follow root conventions. Add per-package files only when a package has unique requirements:

```
packages/
├── ui/
│   ├── AGENTS.md            ← YES: unique (Storybook, a11y rules)
│   └── src/
├── shared/
│   └── src/                 ← NO: follows root conventions
├── api-client/
│   └── src/                 ← NO: follows root conventions
└── config/
    └── src/                 ← NO: follows root conventions
```

**Rule**: Only create per-package AGENTS.md when the package has **different commands, different conventions, or a different tech stack** from the root.

---

## Turborepo / Nx Integration

### Turborepo Task References

```markdown
## Setup
- Run affected tests: `pnpm turbo test --filter=...[HEAD^]`
- Build all: `pnpm turbo build`
- Dev (web): `pnpm turbo dev --filter=web`
```

### Nx Task References

```markdown
## Setup
- Run affected tests: `npx nx affected --target=test`
- Build all: `npx nx run-many --target=build`
- Dev (web): `npx nx serve web`
```

---

## Common Pitfalls in Monorepo AGENTS.md

| Pitfall | Problem | Fix |
|---------|---------|-----|
| Giant root file covering everything | Token waste; diluted rules | Split into per-package files |
| No root file, only package files | No shared rules; boundaries not enforced | Add lean root with global rules |
| Duplicate rules in every package | Maintenance nightmare; drift | Universal rules in root only |
| Cross-package import rules missing | Agent imports between apps | Add boundary: "NEVER import between apps/" |
| Wrong filter/scope in commands | Agent runs wrong package's tests | Use explicit `--filter` or `--scope` flags |
