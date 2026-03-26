# Prompt Files

> Reusable, shareable task-level prompts for common development workflows.

---

## What They Are

Prompt files (`.prompt.md`) are Markdown files stored in `.github/prompts/` that define reusable prompt templates. Unlike instruction files (which are auto-loaded), prompt files are **manually attached** by the developer when needed.

Think of them as **saved workflows** — standardized approaches to common tasks like adding a feature, fixing a bug, or performing a code review.

**Supported by:**
- VS Code (Copilot Chat)
- Visual Studio (Copilot Chat)
- JetBrains IDEs (Copilot Chat)

> ⚠️ Prompt files are currently in **public preview** and subject to change.

---

## When to Use

| Use prompt files for... | Use instruction files for... |
|-------------------------|------------------------------|
| Repeatable task workflows | Always-on conventions |
| Step-by-step procedures | Style and quality rules |
| On-demand templates | Automatic behavior changes |
| Complex multi-step tasks | Background constraints |

---

## Setup

### Enable in VS Code

1. Open **Command Palette** (Ctrl+Shift+P / Cmd+Shift+P)
2. Type "Open Workspace Settings (JSON)"
3. Add:

```json
{
  "chat.promptFiles": true
}
```

This enables the `.github/prompts/` folder as the prompt file location.

### Create a Prompt File

1. **Command Palette** → "Chat: Create Prompt"
2. Enter a name (e.g., `add-feature`)
3. Write your prompt template in Markdown

Or manually create `.github/prompts/add-feature.prompt.md`.

---

## Anatomy of a Prompt File

```markdown
# Add New Feature

Follow this workflow to add a new feature to the application.

## Context
Review the related files to understand current patterns:
- [Component patterns](../../src/components/Button/index.tsx)
- [API route pattern](../../src/app/api/users/route.ts)
- [Test pattern](../../src/components/Button/Button.test.tsx)

## Steps

1. **Plan**: Identify all files that need to change. List them before making changes.
2. **Types**: Define TypeScript interfaces/types first in `src/types/`
3. **Backend**: If API changes are needed, create/update routes in `src/app/api/`
4. **Frontend**: Create/update React components following existing patterns
5. **Tests**: Write tests for all new code (unit + integration)
6. **Validation**: Run `npm run lint && npm run test && npm run build`

## Quality Checklist
- [ ] TypeScript strict — no `any` types
- [ ] All new functions have JSDoc
- [ ] Tests cover happy path and error cases
- [ ] No console.log statements
- [ ] Accessibility: all interactive elements are keyboard-navigable
```

---

## Referencing Other Files

Prompt files can reference project files for additional context:

### Markdown Links (recommended)
```markdown
Review the auth middleware: [auth.ts](../../src/middleware/auth.ts)
```

### Hash-File Syntax
```markdown
Check the current schema: #file:prisma/schema.prisma
```

Paths are **relative to the prompt file**.

---

## Practical Examples

### Bug Fix Workflow

```markdown
# Fix Bug

## Before Starting
- Reproduce the bug locally
- Identify the root cause file(s)
- Check for related tests that should have caught this

## Steps
1. Write a failing test that reproduces the bug
2. Fix the code to make the test pass
3. Check for similar patterns elsewhere that might have the same bug
4. Run the full test suite: `npm run test`
5. Run `npm run lint`

## Commit Message Format
```
fix: [short description]

[Longer explanation of root cause and fix]

Fixes #[issue-number]
```
```

### Code Review Checklist

```markdown
# Code Review

Review the changes against these criteria:

## Correctness
- Does the code do what the PR description says?
- Are edge cases handled?
- Are error states handled gracefully?

## Security
- No hardcoded secrets or credentials
- User input is validated and sanitized
- SQL/NoSQL injection protections in place

## Performance
- No N+1 queries
- Large datasets are paginated
- Heavy computations are memoized or cached

## Maintainability
- Code is self-documenting or properly commented
- No duplicated logic
- Follows existing patterns in the codebase

## Tests
- New code has corresponding tests
- Tests cover both happy and error paths
- No flaky or timing-dependent tests
```

### Database Migration

```markdown
# Database Migration

## Pre-Flight
- Review current schema: #file:prisma/schema.prisma
- Check pending migrations: `npx prisma migrate status`

## Steps
1. Modify `prisma/schema.prisma`
2. Generate migration: `npx prisma migrate dev --name descriptive_name`
3. Review the generated SQL in `prisma/migrations/`
4. Update seed data if needed: #file:prisma/seed.ts
5. Run `npx prisma generate` to update the client
6. Test: `npm run test`

## ⚠️ Production Safety
- New columns must be nullable OR have defaults
- Never drop columns in the same migration that stops using them
- Test rollback: `npx prisma migrate reset` (dev only!)
```

---

## Using Prompt Files

1. Open Copilot Chat
2. Click the **Attach context** icon (📎)
3. Select **Prompt...** from the dropdown
4. Choose your prompt file
5. Optionally add more context or instructions
6. Submit

---

## Best Practices

- ✅ Name files descriptively: `add-feature.prompt.md`, `fix-bug.prompt.md`
- ✅ Include file references for context the AI needs
- ✅ Add validation steps so the AI self-checks
- ✅ Keep prompts focused on one workflow per file
- ✅ Share prompt files across the team via version control
- ❌ Don't put always-on rules here — use `.instructions.md` for that
- ❌ Don't reference files that change frequently (broken references)

---

## Community Examples

GitHub maintains a collection of community-contributed prompt files:
[Awesome GitHub Copilot Customizations](https://github.com/github/awesome-copilot/blob/main/docs/README.prompts.md)

---

*📋 Template: [Prompt File Template](../07-templates/prompt-file-template.md)*

*Next: [AGENTS.md Guide](../03-agents-md/agents-md-guide.md) →*
