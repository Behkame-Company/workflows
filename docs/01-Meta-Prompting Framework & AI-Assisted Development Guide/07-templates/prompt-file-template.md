# Prompt File Template

> Copy and customize. Place in `.github/prompts/` with the `.prompt.md` extension.

---

## Add Feature Prompt

**File**: `.github/prompts/add-feature.prompt.md`

```markdown
# Add New Feature

## Before Starting
- Read [Active Context](../../memory-bank/activeContext.md) for current priorities
- Review [System Patterns](../../memory-bank/systemPatterns.md) for architectural conventions

## Planning
1. **Scope**: List ALL files that will be created or modified
2. **Dependencies**: Identify what must exist before this feature can work
3. **Risks**: Note anything that might break existing functionality
4. **Tests**: Define what tests will prove the feature works correctly

## Implementation Order
1. **Types**: Define TypeScript interfaces/types in `src/types/`
2. **Backend**: Create/update API routes in `src/app/api/` (if needed)
3. **Logic**: Implement business logic in `src/lib/` or `src/services/`
4. **Frontend**: Build UI components following existing patterns
5. **Tests**: Write unit tests for all new code
6. **Integration**: Verify end-to-end with `npm run test:e2e` (if applicable)

## Validation Checklist
- [ ] TypeScript strict — no `any` types
- [ ] All new functions have JSDoc comments
- [ ] Tests cover happy path and error cases
- [ ] No `console.log` statements in production code
- [ ] `npm run lint` passes
- [ ] `npm run test` passes
- [ ] `npm run build` succeeds

## After Completion
- Update `memory-bank/activeContext.md` with the changes made
```

---

## Fix Bug Prompt

**File**: `.github/prompts/fix-bug.prompt.md`

```markdown
# Fix Bug

## Diagnosis
1. Reproduce the bug locally — describe the steps and actual vs. expected behavior
2. Identify the root cause file(s) and function(s)
3. Check for existing tests that should have caught this
4. Look for similar patterns elsewhere that might have the same issue

## Fix Process
1. **Write a failing test** that reproduces the bug
2. **Fix the code** to make the test pass
3. **Check related code** for the same pattern
4. **Run the full test suite**: `npm run test`
5. **Run linting**: `npm run lint`

## Commit Message
```
fix: [short description of the fix]

Root cause: [brief explanation of what was wrong]
Fix: [brief explanation of what changed]

Fixes #[issue-number]
```

## Post-Fix
- Update `memory-bank/activeContext.md` if this affects current work
- Note the pattern in `memory-bank/systemPatterns.md` if it's a new lesson
```

---

## Code Review Prompt

**File**: `.github/prompts/code-review.prompt.md`

```markdown
# Code Review

Review the staged changes against these criteria:

## Correctness
- Does the code do what the PR description says?
- Are edge cases handled (empty input, null, boundaries)?
- Are error states handled gracefully with user-friendly messages?

## Security
- No hardcoded secrets, credentials, or API keys
- User input is validated and sanitized
- SQL injection / XSS protections in place
- Authentication and authorization checks present where needed

## Performance
- No N+1 query patterns
- Large datasets are paginated
- Expensive computations are memoized or cached where appropriate

## Testing
- New code has corresponding test files
- Tests cover both happy path and error cases
- No flaky or timing-dependent tests
- Mocks are used for external services

## Conventions
- Follows patterns in copilot-instructions.md
- Consistent with existing codebase style
- TypeScript types are meaningful (no `any`)
- Components are accessible (semantic HTML, aria labels)

## Output Format
For each issue found, provide:
- **File**: path/to/file.ts
- **Line**: approximate line number
- **Severity**: 🔴 Must Fix / 🟡 Should Fix / 🔵 Suggestion
- **Issue**: What's wrong
- **Fix**: Suggested solution
```

---

## Database Migration Prompt

**File**: `.github/prompts/migrate-db.prompt.md`

```markdown
# Database Migration

## Pre-Flight Checks
- Review current schema: [schema.prisma](../../prisma/schema.prisma)
- Check pending migrations: `npx prisma migrate status`
- Backup: Ensure recent backup exists (production) or reset is acceptable (dev)

## Steps
1. Modify `prisma/schema.prisma` with the schema changes
2. Generate migration: `npx prisma migrate dev --name descriptive_name`
3. Review the generated SQL in `prisma/migrations/[timestamp]_descriptive_name/`
4. Run `npx prisma generate` to update the Prisma Client
5. Update seed data if needed: [seed.ts](../../prisma/seed.ts)
6. Run tests: `npm run test`

## ⚠️ Safety Rules
- New columns MUST be nullable OR have defaults (never required without defaults)
- Never DROP columns in the same migration that stops using them
- For column renames: add new → migrate data → remove old (separate migrations)
- Test rollback in dev: `npx prisma migrate reset`
- Index columns used in WHERE, JOIN, and ORDER BY clauses

## After Migration
- Update `memory-bank/techContext.md` if schema changes are significant
- Notify the team if migration requires manual steps in staging/production
```

---

## Refactoring Prompt

**File**: `.github/prompts/refactor.prompt.md`

```markdown
# Refactoring

## Before Starting
- Ensure all tests pass: `npm run test`
- Create a git checkpoint: `git stash` or commit current work
- Define the goal: what should be better after refactoring?

## Process
1. **Identify**: List the specific code smells or issues to address
2. **Plan**: Map out the changes without modifying code
3. **Test first**: Ensure existing tests cover the code being refactored
4. **Small steps**: Make one change at a time, running tests after each
5. **Verify**: Run the full test suite after all changes

## Rules
- No behavior changes — refactoring must preserve existing functionality
- If tests are missing, write them BEFORE refactoring
- One concern per commit (e.g., "Extract utility function" separate from "Rename variables")
- If the refactoring scope grows, stop and re-plan

## Validation
- `npm run test` — all tests must pass (same tests as before)
- `npm run lint` — no new linting errors
- `npm run build` — must compile
- Manual spot-check: verify the feature still works as expected
```
