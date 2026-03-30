# AGENTS.md Templates Library

> Starter templates you can copy, customize, and commit.

---

## How to Use

1. Choose the template closest to your project
2. Copy the Markdown content
3. Create `AGENTS.md` at your repository root
4. Replace placeholders (`[...]`) with your project's values
5. Commit and push

---

## Template 1: Universal Starter (Any Project)

```markdown
# AGENTS.md

## Setup
- Install: `[your install command]`
- Dev: `[your dev command]`
- Test: `[your test command]`
- Build: `[your build command]`
- Lint: `[your lint command]`

## Stack
[Language], [Framework], [Database], [Test framework].

## Conventions
- [Rule 1: most important coding convention]
- [Rule 2: second most important]
- [Rule 3: third most important]

## Boundaries
- NEVER commit .env files or secrets
- NEVER [your most dangerous operation]
- NEVER [your second most dangerous operation]

## Testing
- Framework: [test framework]
- Run: `[test command]`
- [Testing convention]
```

**Customize**: Replace every `[...]` with your project's specifics.

---

## Template 2: TypeScript Web App

```markdown
# AGENTS.md

## Setup
- Package manager: pnpm (NEVER npm or yarn)
- Install: `pnpm install`
- Dev: `pnpm dev`
- Build: `pnpm build`
- Test: `pnpm test`
- Test single: `pnpm test -- [file]`
- Lint: `pnpm lint`
- Type check: `pnpm typecheck`

## Stack
TypeScript [version], [React/Vue/Svelte] [version], [Next.js/Nuxt/SvelteKit] [version],
[CSS solution], [ORM], [Database].

NOT: [list frameworks/tools you do NOT use]

## Structure
src/app/         — [purpose]
src/components/  — [purpose]
src/lib/         — [purpose]
src/server/      — [purpose]

## Conventions
- TypeScript strict mode; never use `any`
- [Export convention: named/default]
- [Component pattern]
- [Validation library]
- [Error handling pattern]

## Boundaries
- NEVER commit .env files or secrets
- NEVER modify [protected directory]
- NEVER use console.log (use [logger])
- NEVER [import restriction]

## Testing
- Unit: [framework]
- E2E: [framework]
- Run: `[command]`
- Coverage: [target]% for [directories]
```

---

## Template 3: Python Backend

```markdown
# AGENTS.md

## Setup
- Python: [version]+
- Package manager: [uv/pip/poetry]
- Install: `[install command]`
- Dev: `[dev server command]`
- Test: `[test command]`
- Lint: `[lint command]`
- Format: `[format command]`
- Type check: `[type check command]`
- DB migrate: `[migration command]`

## Stack
Python [version], [FastAPI/Django/Flask], [Pydantic/DRF],
[SQLAlchemy/Django ORM], [PostgreSQL/MySQL], [pytest].

NOT: [excluded tools]

## Structure
[app_name]/
├── [routers/views/]     — [purpose]
├── [services/]          — [purpose]
├── [models/]            — [purpose]
├── [schemas/]           — [purpose]
└── [core/]              — [purpose]
tests/                   — [purpose]

## Conventions
- Type hints on all functions
- [Validation pattern]
- [Error handling pattern]
- [Import convention]
- [Naming convention]

## Boundaries
- NEVER use raw SQL (use [ORM])
- NEVER commit .env files
- NEVER modify existing migration files
- NEVER use print() (use [logging solution])
- NEVER [security restriction]

## Testing
- Framework: [pytest/unittest]
- Run: `[command]`
- [Mock strategy]
- [Coverage target]
```

---

## Template 4: Go Service

```markdown
# AGENTS.md

## Setup
- Go: [version]+
- Build: `go build ./...`
- Test: `go test ./... -v`
- Test single: `go test -v -run [TestName] ./[package]`
- Lint: `golangci-lint run`
- Run: `go run cmd/[app]/main.go`

## Stack
Go [version], [router: Chi/Gorilla/stdlib], [DB access: sqlc/sqlx/GORM],
[Database], [testing libs].

NOT: [excluded tools]

## Structure
cmd/[app]/        — Application entry point
internal/
├── handler/      — HTTP handlers
├── service/      — Business logic
├── repository/   — Database access
├── model/        — Domain types
└── middleware/    — HTTP middleware

## Conventions
- Context as first parameter
- Errors returned, never panic
- Interfaces defined by consumers
- Table-driven tests
- [error wrapping pattern]

## Boundaries
- NEVER use panic for error handling
- NEVER use global variables
- NEVER commit .env files
- NEVER modify generated files (modify [source] instead)
```

---

## Template 5: Monorepo Root

```markdown
# AGENTS.md

## Monorepo
[Turborepo/Nx/Lerna] monorepo with [pnpm/yarn] workspaces.
Per-package instructions in their respective AGENTS.md files.

## Setup
- Install: `[install command]` (from root)
- Build all: `[build all command]`
- Test all: `[test all command]`
- Test affected: `[test affected command]`
- Lint all: `[lint all command]`

## Packages
[package 1] — [purpose]
[package 2] — [purpose]
[package 3] — [purpose]

## Global Rules
- Package manager: [pm] (NEVER use [other pm])
- Commits: conventional commits with scope
- Branches: [branch naming pattern]

## Boundaries
- NEVER import between apps/ (use packages/ for sharing)
- NEVER add root-level dependencies
- NEVER commit .env files
- NEVER modify CI/CD without [team] review
```

---

## Template 6: Infrastructure

```markdown
# AGENTS.md

## Setup
- Init: `terraform init`
- Plan: `terraform plan -var-file=environments/[env].tfvars`
- Validate: `terraform validate`
- Format: `terraform fmt -recursive`
- Lint: `tflint --recursive`

## Stack
Terraform [version], [provider] provider [version].

## Structure
modules/          — Reusable modules
environments/     — Per-environment variable files
[main/variables/outputs/providers/backend].tf

## Conventions
- Modules for all reusable infrastructure
- Variables for all configurable values
- Tags on all resources
- Descriptions on all variables and outputs

## Boundaries
- NEVER hardcode account IDs, regions, or IPs
- NEVER store state locally
- NEVER modify prod environment without [team] review
- NEVER run `terraform apply` without reviewing plan
- NEVER commit .terraform/ or .tfstate files
```

---

## After Creating Your AGENTS.md

1. ✅ Commit: `git add AGENTS.md && git commit -m "docs: add AGENTS.md"`
2. ✅ Test: Have an agent try a task
3. ✅ Iterate: Add rules when agents make mistakes
4. ✅ Review: Schedule quarterly audits (see [Audit Checklist](audit-checklist.md))
