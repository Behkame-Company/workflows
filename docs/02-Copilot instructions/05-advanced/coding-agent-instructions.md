# Coding Agent Instructions

> Optimizing instructions specifically for the GitHub Copilot Coding Agent.

---

## How the Coding Agent Differs

The Coding Agent is fundamentally different from Copilot Chat:

| Feature | Copilot Chat | Coding Agent |
|---------|-------------|--------------|
| Mode | Interactive, human-in-the-loop | Autonomous, runs to completion |
| Context | Your open files + conversation | Entire repository + task |
| Output | Suggestions in chat | Pull request with commits |
| Tools | Code search, edit | Code search, edit, terminal commands, file ops |
| Review | Real-time | Async PR review |
| Instruction files | copilot-instructions.md + matching .instructions.md | All of the above + AGENTS.md |

---

## What the Coding Agent Needs

### 1. Commands (Critical)

Without explicit commands, the agent will **guess** how to build and test your project. Often wrong.

```markdown
## Commands
- `npm install` — Install dependencies (always run first)
- `npm run dev` — Start development server
- `npm run test` — Run all tests (agent should run after changes)
- `npm run test:unit` — Run unit tests only (faster)
- `npm run lint` — Check code quality
- `npm run build` — Production build (must pass before PR)
- `npm run db:migrate` — Apply database migrations (if schema changes)
```

**Include**:
- The exact command (not "run the tests")
- When to use it ("always run first", "after changes")
- Prerequisites ("if schema changes")

### 2. Verification Steps (Critical)

Tell the agent how to verify its own work:

```markdown
## Verification
After making changes, verify in this order:
1. `npm run lint` — Fix all errors
2. `npm run test` — All tests must pass
3. `npm run build` — Must compile without errors
4. If you created a new API endpoint, test it with curl
```

Without verification steps, the agent may open a PR with broken code.

### 3. Environment Setup

```markdown
## Environment
- Node.js 20+ required
- pnpm as package manager (not npm or yarn)
- PostgreSQL database (connection string in .env)
- Run `pnpm install` before any other command
- Run `pnpm db:push` to sync database schema
```

### 4. Scope Boundaries

```markdown
## Scope
- You may create new files in src/
- You may modify existing files in src/
- Do NOT modify: .env, package.json (dependencies), CI configs
- Do NOT delete any existing files
- If the task requires adding a dependency, note it in the PR description
```

---

## Coding Agent-Specific Patterns

### Pattern: Step-by-Step Workflow

The Coding Agent follows sequential instructions well:

```markdown
## Workflow for New Features
1. Read relevant existing code to understand patterns
2. Create necessary files following existing conventions
3. Write the implementation
4. Add or update tests
5. Run `npm run test` to verify
6. Run `npm run lint` to check code quality
7. Run `npm run build` to verify compilation
```

### Pattern: PR Description Template

```markdown
## Pull Request Standards
When creating a PR, include:
- Summary of changes
- List of files created/modified
- Test results (paste output)
- Any manual testing needed
- Breaking changes (if any)
```

### Pattern: Task Decomposition

For complex tasks, guide the agent's approach:

```markdown
## Complex Tasks
For tasks involving multiple components:
1. Break the task into smaller subtasks
2. Implement one subtask at a time
3. Test each subtask before moving to the next
4. Commit logical units of work separately
```

---

## Common Coding Agent Failures

### Failure: Wrong Package Manager

```
Agent runs: npm install
Project uses: pnpm

→ Creates package-lock.json (wrong lockfile)
→ Dependencies may resolve differently
→ CI fails
```

**Fix**: Explicitly state the package manager:

```markdown
## Commands
Package manager: pnpm (never use npm or yarn)
```

### Failure: Skips Testing

```
Agent makes changes
Agent creates PR without running tests
Tests fail in CI
```

**Fix**: Make testing a required verification step:

```markdown
## Verification
ALWAYS run `pnpm test` before creating the PR.
If tests fail, fix the code and rerun.
Do not create a PR with failing tests.
```

### Failure: Modifies Protected Files

```
Agent modifies .env, CI configs, or core dependency list
Changes break deployment or security
```

**Fix**: Explicit scope boundaries (see above).

### Failure: Incorrect Test Patterns

```
Agent writes tests using Jest patterns
Project uses Vitest
Tests don't run
```

**Fix**: Specify the test framework and patterns:

```markdown
## Testing
Framework: Vitest (not Jest)
Test file naming: `*.test.ts` co-located with source
Pattern: describe → it → Arrange/Act/Assert
Mock external services with `vi.mock()`
```

---

## AGENTS.md Synergy

For the Coding Agent, consider maintaining both files:

```markdown
# AGENTS.md (cross-tool, also read by Claude Code/Codex)
[Project description, commands, conventions]

# .github/copilot-instructions.md (Copilot-specific extras)
See AGENTS.md for project conventions.

## Copilot Coding Agent Specifics
- Always run full test suite before PR
- Include test output in PR description
- Commit granularly (one logical change per commit)
```

---

## Instruction Tuning for Agent Quality

### Improve Code Quality

```markdown
## Code Quality
- Match the style of surrounding code
- Follow existing naming patterns in the module
- Add JSDoc comments for new public functions
- Prefer small, focused functions over large ones
```

### Improve Commit Quality

```markdown
## Commits
- Use conventional commits: feat:, fix:, docs:, test:, refactor:
- One logical change per commit
- Write clear commit messages explaining WHY, not just WHAT
```

### Improve PR Quality

```markdown
## Pull Requests
- Title follows: [type]: [short description]
- Description includes: summary, changes list, testing notes
- Reference the GitHub issue: "Closes #123"
- Flag any areas needing human review
```
