# Prompt Files: .prompt.md

> Reusable, on-demand workflow templates for Copilot Chat.

---

## Overview

| Property | Value |
|----------|-------|
| **Location** | `.github/prompts/NAME.prompt.md` |
| **Scope** | On-demand — user manually triggers |
| **Auto-loaded** | No — must be invoked or attached |
| **Format** | Optional YAML frontmatter + Markdown body |
| **Supported by** | Copilot Chat (VS Code, Visual Studio, JetBrains) |
| **Requires** | VS Code setting: `"chat.promptFiles": true` |

---

## What Makes Prompt Files Different

| Feature | Instructions | Prompt Files |
|---------|-------------|--------------|
| When loaded | Automatically | On-demand |
| Purpose | Rules/constraints | Task workflows |
| Behavior | "Always do this" | "Do this now" |
| Analogy | Team coding standards | Run this recipe |
| Reusability | Persistent context | Triggered templates |

Think of instructions as **passive rules** and prompts as **active recipes**.

---

## File Format

### Minimal Prompt

```markdown
Generate unit tests for the selected code using Vitest.
Include edge cases, error scenarios, and happy path tests.
Use the Arrange-Act-Assert pattern.
```

### Prompt with Frontmatter

```markdown
---
name: generate-tests
description: Create comprehensive unit tests for selected code
agent: agent
tools:
  - search
  - edit
  - run
model: gpt-4
---
# Generate Tests

Analyze the selected code and generate comprehensive tests:

1. Happy path tests for all public functions
2. Edge case tests (null, undefined, empty, boundary values)
3. Error handling tests (invalid input, exceptions)
4. Integration tests if the code interacts with external services

Use Vitest and React Testing Library.
Follow the Arrange → Act → Assert pattern.
Name tests: `it('should [behavior] when [condition]')`
```

### Frontmatter Fields

| Field | Purpose |
|-------|---------|
| `name` | Display name in Copilot's prompt picker |
| `description` | Short description shown in the picker |
| `agent` | Force agent mode (`agent`) or chat mode |
| `tools` | Which tools the agent can use (`search`, `edit`, `run`, `fetch`) |
| `model` | Preferred model for this prompt |

---

## How to Use Prompt Files

### Method 1: Chat Command

In Copilot Chat, type `/` and select your prompt from the list.

### Method 2: Attach to Chat

Click the paperclip icon in Copilot Chat → Prompts → select the prompt file.

### Method 3: Reference in Chat

Type `#prompt:generate-tests` in your chat message to include it as context.

---

## Prompt File Patterns

### Pattern 1: Feature Implementation

```markdown
---
name: add-feature
description: Structured feature implementation workflow
agent: agent
tools: [search, edit, run]
---
# Add Feature

## Objective
Implement the feature described below following our project conventions.

## Steps
1. **Understand**: Read related existing code with `search`
2. **Plan**: List files to create/modify before coding
3. **Implement**: Write the code following project conventions
4. **Test**: Create unit tests and run them with `run`
5. **Verify**: Run `npm run lint && npm run build` to validate

## Conventions to Follow
- See #file:.github/copilot-instructions.md for project standards
- TypeScript strict mode, named exports, Zod validation

## My Feature Request
[User fills in their specific feature here]
```

### Pattern 2: Bug Investigation

```markdown
---
name: investigate-bug
description: Systematic bug investigation and fix
agent: agent
tools: [search, edit, run]
---
# Bug Investigation

1. **Reproduce**: Identify the failing behavior from the description
2. **Locate**: Search the codebase for the relevant code path
3. **Analyze**: Read the code and understand the root cause
4. **Fix**: Implement the minimal fix
5. **Test**: Write a regression test that would have caught this
6. **Verify**: Run `npm run test` to confirm fix and no regressions

## Bug Description
[User describes the bug here]
```

### Pattern 3: Code Review Prep

```markdown
---
name: review-prep
description: Pre-review checklist for changes
---
# Pre-Review Checklist

Review my recent changes and verify:

- [ ] No `console.log` or debugging artifacts left
- [ ] All new functions have JSDoc comments
- [ ] No hardcoded values that should be config/env
- [ ] Error handling is complete (no unhandled promises)
- [ ] Types are explicit (no implicit `any`)
- [ ] Tests exist for new logic
- [ ] No unused imports
- [ ] Consistent naming conventions
```

### Pattern 4: Documentation Generator

```markdown
---
name: generate-docs
description: Generate API documentation for a module
agent: agent
tools: [search]
---
# Generate Documentation

For the file or module I specify:

1. Read all exported functions, types, and classes
2. Generate comprehensive JSDoc for each
3. Create a module-level overview comment
4. List all dependencies and their purpose
5. Note any side effects or important behaviors
6. Format as clean markdown documentation
```

### Pattern 5: Database Migration

```markdown
---
name: create-migration
description: Safe database migration workflow
agent: agent
tools: [search, edit, run]
---
# Create Database Migration

## Safety Rules
- New columns on existing tables MUST be nullable or have defaults
- Never modify or delete existing columns
- Always add indexes for foreign keys
- Test migration is reversible

## Steps
1. Understand what schema change is needed
2. Modify `prisma/schema.prisma`
3. Run `npx prisma migrate dev --name [descriptive-name]`
4. Verify the generated SQL in `prisma/migrations/`
5. Test with `npm run test`

## Migration Request
[User describes the schema change here]
```

### Pattern 6: Refactoring

```markdown
---
name: refactor-code
description: Structured refactoring with safety checks
agent: agent
tools: [search, edit, run]
---
# Refactor Code

## Safety Protocol
1. Run tests BEFORE making changes: `npm run test`
2. Make one logical change at a time
3. Run tests AFTER each change
4. If tests fail, revert and try a different approach

## Refactoring Techniques
- Extract Method: Pull logic into named functions
- Extract Component: Break large components into smaller ones
- Rename: Use clear, descriptive names
- Reduce Duplication: DRY without over-abstracting

## What to Refactor
[User describes what needs refactoring and why]
```

---

## Context References in Prompts

Prompt files can reference other files for context:

```markdown
# Coding Standards Reference
Follow the conventions in #file:.github/copilot-instructions.md

# Architecture Reference
Understand the structure from #file:docs/architecture.md

# Example Code
Follow the pattern in #file:src/components/ExampleComponent.tsx
```

**Supported references:**
- `#file:path/to/file` — Include file contents
- `#selection` — Currently selected code in editor
- `#codebase` — Repository-wide search context

---

## Organizing Prompt Files

### Recommended Structure

```
.github/prompts/
├── add-feature.prompt.md        # Feature implementation
├── fix-bug.prompt.md             # Bug investigation
├── generate-tests.prompt.md      # Test generation
├── create-migration.prompt.md    # DB migration
├── review-prep.prompt.md         # Pre-review checklist
├── refactor.prompt.md            # Code refactoring
├── generate-docs.prompt.md       # Documentation
├── performance-audit.prompt.md   # Performance review
└── onboarding.prompt.md          # New developer guide
```

### Naming Convention

`[verb]-[noun].prompt.md`
- `add-feature.prompt.md` ✅
- `fix-bug.prompt.md` ✅
- `generate-tests.prompt.md` ✅
- `my-prompt.prompt.md` ❌ (not descriptive)
- `stuff.prompt.md` ❌ (meaningless)

---

## Tips for Effective Prompts

1. **Include the "why"** — explain what success looks like
2. **Number the steps** — sequential instructions are followed more reliably
3. **Reference conventions** — link to your instruction files
4. **Include verification** — end with "run tests" or "verify build"
5. **Leave placeholders** — `[User describes...]` for user-specific input
6. **Use agent mode** — for multi-step tasks that need tool access
7. **Keep focused** — one prompt = one workflow (don't combine unrelated tasks)
