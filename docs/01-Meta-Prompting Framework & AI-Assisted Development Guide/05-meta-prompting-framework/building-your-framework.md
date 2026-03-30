# Building Your Framework

> A step-by-step guide to implementing a meta-prompting framework from scratch.

---

## Prerequisites

- A Git repository with an existing or new project
- GitHub Copilot (Business or Enterprise recommended for team features)
- VS Code with Copilot Chat extension
- Team buy-in on adopting structured AI instructions

---

## Phase 1: Foundation (Day 1)

### Step 1: Create copilot-instructions.md

This is your highest-impact starting point.

```bash
mkdir -p .github
```

Create `.github/copilot-instructions.md` with these essential sections:

```markdown
# Repository Instructions

## Project
[2-3 sentence description of your project, stack, and current status]

## Commands
- `npm install` — Install dependencies
- `npm run dev` — Start development server
- `npm run build` — Production build
- `npm run test` — Run all tests
- `npm run lint` — Lint check

Always run lint and tests before considering work complete.

## Conventions
[5-10 bullet points of your most important coding rules]

## Do Not
[3-5 explicit boundaries]

## Validation
After changes, run: `npm run lint && npm run test && npm run build`
```

💡 **Tip**: Start lean. You can always add more. A focused half-page beats a bloated 3 pages.

### Step 2: Create AGENTS.md

Create `AGENTS.md` at the repo root with your universal rules:

```markdown
# AGENTS.md

## Setup
- Install: `npm install`
- Dev: `npm run dev`
- Test: `npm run test`
- Lint: `npm run lint`
- Build: `npm run build`

## Code Style
[Your most important conventions — shared across all AI tools]

## Do / Don't
[Universal boundaries any AI agent should respect]
```

### Step 3: Test It

1. Open Copilot Chat in VS Code
2. Ask it to do something your instructions address (e.g., "Create a new component")
3. Check the References list to confirm `copilot-instructions.md` was loaded
4. Verify the output follows your conventions

---

## Phase 2: Domain Specialization (Week 1)

### Step 4: Identify Domain Boundaries

Map your codebase into instruction domains:

| Domain | Path Pattern | Key Rules |
|--------|-------------|-----------|
| Frontend components | `src/components/**/*.tsx` | React patterns, Tailwind, accessibility |
| API routes | `src/app/api/**/*.ts` | Validation, error handling, status codes |
| Database | `prisma/**/*` | Migration safety, schema patterns |
| Tests | `**/*.test.*` | Test patterns, mocking, assertions |

### Step 5: Create Path-Specific Instructions

```bash
mkdir -p .github/instructions
```

Create one `.instructions.md` file per domain:

```markdown
---
applyTo: "src/components/**/*.tsx,src/components/**/*.ts"
---

# Frontend Component Rules
[Domain-specific conventions for this path]
```

### Step 6: Create Your First Prompt File

```bash
mkdir -p .github/prompts
```

Start with a standard feature workflow:

```markdown
# Add Feature

## Steps
1. Plan: identify all files that need to change
2. Types: define interfaces first
3. Implementation: follow existing patterns
4. Tests: write tests for all new code
5. Validation: run lint, tests, and build

## References
- [Component patterns](../../src/components/Button/index.tsx)
```

Enable prompt files in VS Code:

```json
{ "chat.promptFiles": true }
```

---

## Phase 3: Memory (Week 2)

### Step 7: Initialize Memory Bank

```bash
mkdir -p memory-bank
```

Create the core files — start with `projectbrief.md` and `activeContext.md`, then add others as needed:

| File | Purpose |
|------|---------|
| `projectbrief.md` | Stable project scope and goals |
| `activeContext.md` | Current sprint focus, recent decisions, next steps |
| `systemPatterns.md` | Architecture decisions and design patterns |
| `techContext.md` | Stack details and environment setup |
| `productContext.md` | Problem statement and user personas |
| `progress.md` | Milestone tracking and known issues |

See [Memory Bank for Teams](../../04-memory-bank/memory-bank-for-teams.md) for detailed file descriptions and the [Memory Bank Template](../../07-templates/memory-bank-template.md) for ready-to-copy templates.

### Step 8: Link Memory to Instructions

Add a reference in `copilot-instructions.md`:

```markdown
## Project Context
For current work status and architecture decisions, read
memory-bank/activeContext.md before starting complex tasks.
```

### Step 9: Establish the Update Cadence

| Event | Action |
|-------|--------|
| Sprint start | Update `activeContext.md` with sprint goals |
| Feature complete | Move from "In Progress" to "Recently Completed" |
| Architecture decision | Add to `systemPatterns.md` |
| Sprint end | Archive completed items, update `progress.md` |
| Session end | Ask AI to update `activeContext.md` with current state |

---

## Phase 4: Refinement (Week 3+)

### Step 10: Add Remaining Memory Bank Files

As the team uses the framework, add files as needed:

- `systemPatterns.md` — after architectural patterns emerge
- `techContext.md` — when setup complexity requires documentation
- `productContext.md` — when product context matters for AI decisions
- `progress.md` — when milestone tracking becomes valuable

### Step 11: Create More Prompt Files

Based on your team's most common workflows:

| Workflow | Prompt File |
|----------|-------------|
| Standard feature | `add-feature.prompt.md` |
| Bug fix | `fix-bug.prompt.md` |
| Database migration | `migrate-db.prompt.md` |
| Code review prep | `code-review.prompt.md` |
| Refactoring | `refactor.prompt.md` |

### Step 12: Iterate Based on Feedback

Collect signals from the team:

| Signal | Action |
|--------|--------|
| AI keeps making the same mistake | Add explicit rule to instructions |
| AI asks the same question every session | Add answer to memory bank |
| Different team members get inconsistent results | Tighten instruction specificity |
| Instructions file is too long | Split into path-specific files |
| AI ignores instructions | Check that file is in correct location and format |

---

## Phase 5: Optimization (Ongoing)

### Step 13: Quarterly Instruction Review

Every quarter, audit your instruction files:

1. **Remove stale entries** — outdated conventions, resolved issues
2. **Consolidate duplicates** — merge repeated rules
3. **Test compliance** — verify the AI actually follows instructions
4. **Benchmark** — compare AI output quality before/after instruction changes
5. **Right-size** — if any file exceeds 2 pages, split or compress

### Step 14: Measure Impact

Track these metrics to justify the framework:

| Metric | How to Measure |
|--------|----------------|
| PR review cycles | Fewer revision requests from reviewers? |
| Convention violations | Fewer style/pattern issues in PRs? |
| Onboarding time | New devs productive faster with AI? |
| AI session length | Less time re-explaining context? |
| Build failures | Fewer CI failures from AI-generated code? |

---

## Complete File Tree After Setup

```
your-project/
├── .github/
│   ├── copilot-instructions.md
│   ├── instructions/
│   │   ├── frontend.instructions.md
│   │   ├── api.instructions.md
│   │   ├── database.instructions.md
│   │   └── testing.instructions.md
│   └── prompts/
│       ├── add-feature.prompt.md
│       ├── fix-bug.prompt.md
│       └── code-review.prompt.md
├── AGENTS.md
├── memory-bank/
│   ├── projectbrief.md
│   ├── productContext.md
│   ├── systemPatterns.md
│   ├── techContext.md
│   ├── activeContext.md
│   └── progress.md
└── ... (your code)
```

---

## Common Pitfalls

| Pitfall | Solution |
|---------|----------|
| Writing instructions once and never updating | Schedule quarterly reviews |
| Making instructions too long | Keep each file under 2 pages |
| Not testing that instructions work | Verify after every change |
| Everyone editing memory bank without coordination | Assign file ownership |
| Trying to adopt everything at once | Follow the phased approach above |
