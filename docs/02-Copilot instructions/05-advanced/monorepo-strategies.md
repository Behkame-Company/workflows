# Monorepo Instruction Strategies

> Managing Copilot instructions across polyglot and multi-project repositories.

---

## The Monorepo Challenge

Monorepos contain multiple projects, languages, and teams. A single copilot-instructions.md cannot serve all contexts effectively.

```
monorepo/
├── apps/
│   ├── web/          # Next.js frontend
│   ├── mobile/       # React Native app
│   └── admin/        # Internal dashboard
├── packages/
│   ├── ui/           # Shared component library
│   ├── api-client/   # Generated API client
│   └── config/       # Shared configurations
├── services/
│   ├── api/          # Express.js API
│   ├── workers/      # Background job processors
│   └── gateway/      # API gateway
└── infrastructure/
    ├── terraform/    # Cloud infrastructure
    └── k8s/          # Kubernetes configs
```

---

## Strategy 1: Thin Root + Heavy Path-Specific

### Root: Minimal Universal Rules

```markdown
# .github/copilot-instructions.md (150 words max)

## Project
Monorepo for [product name]. Turborepo + pnpm workspaces.

## Commands (Root)
- `pnpm install` — Install all workspace dependencies
- `pnpm build` — Build all packages
- `pnpm lint` — Lint all packages
- `pnpm test` — Run all tests

## Universal Conventions
- TypeScript strict mode in all packages
- Named exports only
- Conventional commit messages

## Do Not
- Never modify package.json dependencies without team discussion
- Never bypass TypeScript strict checks
```

### Path-Specific: Domain Details

```
.github/instructions/
├── web-app.instructions.md        # applyTo: "apps/web/**"
├── mobile-app.instructions.md     # applyTo: "apps/mobile/**"
├── api-service.instructions.md    # applyTo: "services/api/**"
├── shared-ui.instructions.md      # applyTo: "packages/ui/**"
├── workers.instructions.md        # applyTo: "services/workers/**"
├── infrastructure.instructions.md # applyTo: "infrastructure/**"
└── testing.instructions.md        # applyTo: "**/*.test.{ts,tsx}"
```

---

## Strategy 2: AGENTS.md Per Directory

Use AGENTS.md files at each project boundary:

```
monorepo/
├── AGENTS.md                     # Universal rules
├── apps/
│   ├── web/
│   │   └── AGENTS.md             # Web-specific rules
│   └── mobile/
│       └── AGENTS.md             # Mobile-specific rules
├── services/
│   ├── api/
│   │   └── AGENTS.md             # API service rules
│   └── workers/
│       └── AGENTS.md             # Worker rules
└── packages/
    └── ui/
        └── AGENTS.md             # UI library rules
```

**Pros**: Works with Copilot Coding Agent, Claude Code, and Cursor simultaneously.
**Cons**: Many files to maintain; tool support for hierarchy varies.

---

## Strategy 3: Package-Scoped Commands

The most critical monorepo instruction: **which commands to run and where**.

```markdown
# .github/instructions/web-app.instructions.md
---
applyTo: "apps/web/**"
---
## Commands (apps/web)
- `pnpm --filter @org/web dev` — Start web dev server
- `pnpm --filter @org/web test` — Run web tests
- `pnpm --filter @org/web build` — Build web app
- `cd apps/web && pnpm test` — Alternative: run from package directory

## Stack
Next.js 15 with App Router, Tailwind CSS, Zustand for state.
```

```markdown
# .github/instructions/api-service.instructions.md
---
applyTo: "services/api/**"
---
## Commands (services/api)
- `pnpm --filter @org/api dev` — Start API server
- `pnpm --filter @org/api test` — Run API tests
- `pnpm --filter @org/api build` — Build API

## Stack
Express.js, Prisma ORM, PostgreSQL.
```

---

## Glob Patterns for Monorepo Structures

| Pattern | Matches |
|---------|---------|
| `"apps/web/**"` | Everything in the web app |
| `"apps/**/*.tsx"` | React components in all apps |
| `"services/**/*.ts"` | TypeScript files in all services |
| `"packages/ui/src/**"` | UI library source code |
| `"**/*.test.{ts,tsx}"` | Test files everywhere |
| `"infrastructure/**"` | All infrastructure configs |
| `"**/prisma/**"` | Prisma files in any package |

---

## Cross-Package Conventions

### Shared Types

```markdown
# In shared.instructions.md (applyTo: "packages/**")
## Shared Package Rules
- Shared packages in packages/ export types and utilities
- Never import from apps/ or services/ (dependency direction: packages → apps)
- All exports must be listed in the package's `index.ts`
- Changes to shared packages require running tests in all consuming apps
```

### Import Boundaries

```markdown
## Import Rules
- apps/web imports from: packages/*, node_modules
- services/api imports from: packages/*, node_modules
- packages/* imports from: other packages/*, node_modules
- NEVER import between apps/ and services/
- NEVER import from apps/ into packages/
```

---

## Turborepo / Nx Integration

### Build Dependencies

```markdown
## Build Order
Turborepo manages build order. Individual package commands:
- `turbo run build --filter=@org/web` — Build web + its dependencies
- `turbo run test --filter=@org/api...` — Test API + all dependencies

When changing shared packages, run: `turbo run test --filter=...@org/changed-package`
to test all downstream consumers.
```

---

## Common Monorepo Pitfalls

| Pitfall | Why It's Bad | Fix |
|---------|-------------|-----|
| One instruction file for entire monorepo | Web devs get API rules, API devs get web rules | Use path-specific files |
| Missing package-specific commands | Agent runs `npm test` at root (maybe wrong) | Include `--filter` commands per package |
| Cross-package import violations | Creates circular dependencies | Document import boundaries explicitly |
| Global "Do Not" rules that are too broad | "Never use default exports" breaks Next.js pages | Make domain-specific exceptions |
| No shared convention documentation | Each package reinvents patterns | Document shared conventions in root file |

---

*Next: [Enterprise & Org Strategies](enterprise-strategies.md) →*
