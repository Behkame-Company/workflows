# copilot-instructions.md Guide

> The single most impactful file for AI-assisted development with GitHub Copilot.

---

## What It Is

`copilot-instructions.md` is a Markdown file placed at `.github/copilot-instructions.md` in your repository. It provides Copilot with repository-specific guidance that is automatically injected into every request made in the context of that repo.

**Supported by:**
- Copilot Chat (VS Code, Visual Studio, JetBrains, GitHub.com)
- Copilot Coding Agent
- Copilot Code Review

---

## Quick Setup

```bash
# Create the directory and file
mkdir -p .github
touch .github/copilot-instructions.md
```

Add your instructions in natural language Markdown. They take effect immediately upon saving.

---

## What to Include

Structure your instructions with these sections (in priority order):

### 1. Project Overview (2–3 sentences)

```markdown
## Project
This is a Next.js 15 SaaS dashboard for marketing analytics.
Stack: TypeScript, React 18, Tailwind CSS, Prisma, PostgreSQL.
Currently in production with active development.
```

💡 Keep this brief — the AI can discover details from your code. Focus on what's not obvious.

### 2. Build, Test, and Lint Commands

```markdown
## Commands
- `npm run dev` — Start development server
- `npm run build` — Production build (must pass before PR)
- `npm run test` — Run all tests (Jest + React Testing Library)
- `npm run test:e2e` — Cypress end-to-end tests
- `npm run lint` — ESLint + Prettier check
- `npm run db:migrate` — Apply Prisma migrations
- `npm run db:seed` — Seed development database

Always run `npm install` before building if dependencies have changed.
Always run `npm run lint` and `npm run test` before considering work complete.
```

⚠️ **This is the highest-impact section.** Without it, the AI will guess commands, often incorrectly.

### 3. Code Conventions

```markdown
## Conventions
- TypeScript strict mode; never use `any`
- Named exports only; no default exports
- Use `async/await`, not `.then()` chains
- All components in `src/components/` use the pattern: `ComponentName/index.tsx` + `ComponentName.test.tsx`
- API routes follow REST conventions in `src/app/api/`
- Use Zod for all input validation
- All functions handling user data must have JSDoc comments
```

💡 Be specific and actionable. "Write clean code" is not useful. "Use named exports" is.

### 4. Project Structure

```markdown
## Structure
src/
  app/          # Next.js App Router pages and API routes
  components/   # Reusable React components
  lib/          # Utilities, helpers, and business logic
  types/        # Shared TypeScript type definitions
  db/           # Prisma schema and migrations
  hooks/        # Custom React hooks
```

### 5. Boundaries and Safety

```markdown
## Do Not
- Never commit secrets or API keys; use environment variables
- Never modify files in `legacy/` or `deprecated/`
- Never use `console.log` in production code; use `lib/logger.ts`
- Never disable TypeScript errors with @ts-ignore
- Never install new dependencies without team approval
```

### 6. Validation Steps

```markdown
## Validation
After making changes:
1. Run `npm run lint` — fix all errors
2. Run `npm run test` — all tests must pass
3. Run `npm run build` — must compile without errors
4. If UI changes: verify against designs in `/docs/designs/`
5. If API changes: update the OpenAPI spec in `/docs/api/`
```

---

## Writing Effective Instructions

### ✅ Good Patterns

| Pattern | Example |
|---------|---------|
| **Imperative voice** | "Always run tests before committing" |
| **Specific tools** | "Use Zod for validation, not Joi" |
| **Concrete paths** | "Components live in `src/components/`" |
| **Explicit commands** | "`npm run test -- --watch` for development" |
| **Known pitfalls** | "The auth module requires `NEXT_PUBLIC_AUTH_URL` to be set" |

### ❌ Anti-Patterns

| Anti-Pattern | Why It Fails |
|-------------|-------------|
| "Write clean, maintainable code" | Too vague; not actionable |
| Copy-pasting the entire README | Wastes context budget |
| 500+ lines of instructions | AI attention degrades |
| Task-specific instructions | Use `.prompt.md` files instead |
| Outdated architecture decisions | AI acts on stale info |

---

## Verification

To confirm your instructions are being used:

1. **In VS Code**: Open Copilot Chat → make a request → expand the **References** list at the top of the response → look for `copilot-instructions.md`
2. **In GitHub.com Chat**: Attach the repo → check References after getting a response
3. **Test behaviorally**: Ask Copilot to do something your instructions explicitly forbid or require — verify it follows the rules

---

## Auto-Generation

You can ask the Copilot Coding Agent to generate a `copilot-instructions.md` for an existing repo:

1. Go to [github.com/copilot/agents](https://github.com/copilot/agents)
2. Select your repository
3. Use the onboarding prompt from [GitHub's documentation](https://docs.github.com/en/copilot/how-tos/configure-custom-instructions/add-repository-instructions)
4. Copilot will create a PR with generated instructions

💡 **Recommended**: Auto-generate first, then manually refine. The auto-generated version captures build/test commands well but may miss team conventions.

---

## Maintenance

| Trigger | Action |
|---------|--------|
| Major dependency upgrade | Update commands and version references |
| Architecture refactor | Update project structure section |
| New team convention | Add to conventions section |
| Quarterly review | Remove stale entries, verify commands still work |
| New team member onboarded | Use their confusion as signal for missing instructions |

---

## Size Guidelines

- **Target**: 1–2 pages of Markdown (roughly 500–1000 words)
- **Maximum**: Don't exceed 2 pages — diminishing returns past this point
- **If you need more**: Split into path-specific `.instructions.md` files

---

*📋 Template: [copilot-instructions.md Template](../07-templates/copilot-instructions-template.md)*

*Next: [Path-Specific Instructions](path-specific-instructions.md) →*
