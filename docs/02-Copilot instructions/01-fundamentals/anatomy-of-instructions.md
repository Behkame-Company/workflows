# Anatomy of a Copilot Instructions File

> Understanding the structure, syntax, and how each section influences Copilot's behavior.

---

## File Format

All Copilot instruction files use **Markdown** (.md). The content is natural language — no special syntax, no YAML configuration (except for path-specific files which use frontmatter).

```markdown
# Section Heading

- Bullet point instruction
- Another instruction

## Subsection

Paragraph of context or explanation.

`code snippets` are supported for commands or patterns.
```

---

## Recommended Section Order

Copilot processes content **top-down**. Place the highest-impact information first:

```markdown
1. ## Project          ← What is this? (2-3 sentences)
2. ## Commands         ← How to build/test/lint (most critical)
3. ## Structure        ← Where things live
4. ## Conventions      ← Coding rules
5. ## Do Not           ← Explicit boundaries
6. ## Testing          ← Test patterns
7. ## Validation       ← Self-check steps
```

### Why Order Matters

AI models have **recency bias** (attention to the end) and **primacy bias** (attention to the beginning). The middle of a long file gets the least attention. Strategy:

- **Top**: Commands and critical rules (primacy)
- **Middle**: Structural context and conventions
- **Bottom**: Validation steps and reminders (recency)

---

## Section-by-Section Breakdown

### Project Section

**Purpose**: Give Copilot a mental model of what this codebase is.

```markdown
## Project
Marketing analytics SaaS dashboard.
Stack: Next.js 15, TypeScript, Tailwind CSS, Prisma, PostgreSQL.
Status: Production, active feature development.
```

**Rules:**
- Keep to 2-3 sentences
- Mention the stack explicitly (don't make Copilot guess)
- Include current status (production vs. prototype)

### Commands Section

**Purpose**: Tell Copilot exactly how to build, test, and validate code.

```markdown
## Commands
- `npm install` — Install dependencies (run after pulling changes)
- `npm run dev` — Development server (http://localhost:3000)
- `npm run build` — Production build (must pass before PR)
- `npm run test` — Run all tests
- `npm run lint` — ESLint + Prettier check
- `npm run db:migrate` — Apply database migrations
```

**Rules:**
- Include the exact command, not just "run the tests"
- Add expected behavior or URLs where helpful
- Note prerequisites ("always run install first")
- This is the **single most important section** — include it even if you skip everything else

### Structure Section

**Purpose**: Help Copilot find things without exploring.

```markdown
## Structure
src/
  app/          # Next.js App Router
  components/   # React components
  lib/          # Utilities
  types/        # TypeScript types
  hooks/        # Custom hooks
prisma/         # Database schema and migrations
```

**Rules:**
- Only include top-level directories and their purpose
- Don't list every file — that wastes context budget
- Call out non-obvious structure (e.g., "tests are co-located, not in a separate folder")

### Conventions Section

**Purpose**: Define the coding standards Copilot must follow.

```markdown
## Conventions
- TypeScript strict mode; never use `any`
- Named exports only; no default exports
- Functional components with explicit prop interfaces
- Use `async/await`, not `.then()` chains
- All API inputs validated with Zod
```

**Rules:**
- Use imperative voice ("Use X", not "We prefer X")
- Be specific and actionable
- Include the **tool or library** to use, not just the concept
- Limit to 5-10 rules; more than that dilutes attention

### Do Not Section

**Purpose**: Explicit boundaries the AI must never cross.

```markdown
## Do Not
- Never commit secrets or API keys
- Never modify files in `legacy/` or `generated/`
- Never disable TypeScript strict checks
- Never add dependencies without team approval
- Never use `console.log` in production code
```

**Rules:**
- Use "Never" for absolute rules
- Be specific about what's forbidden and where
- This creates **negative constraints** which are often more effective than positive rules

### Validation Section

**Purpose**: Tell Copilot how to verify its own work.

```markdown
## Validation
After making changes:
1. `npm run lint` — fix all errors
2. `npm run test` — all tests must pass
3. `npm run build` — must compile without errors
```

**Rules:**
- Include specific commands, not just "run tests"
- Order from fastest to slowest (lint → test → build)
- This closes the feedback loop — Copilot can self-check

---

## Formatting Tips

### ✅ Effective Formatting

```markdown
## Conventions
- Use TypeScript strict mode
- Named exports only
- Validate API input with Zod
```

### ❌ Ineffective Formatting

```markdown
When writing code, please try to use TypeScript in strict mode whenever 
possible. We generally prefer named exports over default exports, though 
there are some exceptions. For API endpoints, it would be nice if you 
could validate input using Zod, but other methods are acceptable too.
```

**Why?** Bullets are parsed more reliably. Hedging language ("try to", "whenever possible") creates ambiguity.

---

## Character Limits

| Context | Approximate Limit |
|---------|-------------------|
| Copilot Chat context | ~8,000-16,000 tokens total |
| Code Review instructions | ~4,000 characters processed |
| Recommended instruction file size | 500-1,000 words (1-2 pages) |

⚠️ Exceeding these limits means Copilot **silently ignores** content beyond the window. Put critical rules first.

---

*Next: [Instruction Hierarchy](instruction-hierarchy.md) →*
