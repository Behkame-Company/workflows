# Phase 4: Implement

> Executing tasks against the specification with AI-assisted validation.

---

## Purpose

The Implement phase is where code gets written. In SDD, implementation is **guided** and **validated** by the artifacts from previous phases:

```
spec.md   → Defines WHAT success looks like
plan.md   → Defines HOW we're building it
tasks.md  → Defines the ORDER of work
code      → The deliverable, validated against all three
```

---

## Implementation Workflow

```
For each task in tasks.md:
  ┌──────────────────────────────────┐
  │ 1. Read task requirements        │
  │ 2. Set up context (spec + plan)  │
  │ 3. Generate or write code        │
  │ 4. Run tests                     │
  │ 5. Validate against acceptance   │
  │ 6. Commit with spec reference    │
  │ 7. Mark task complete            │
  └──────────────────────────────────┘
```

---

## Implementation with AI Agents

### Context Loading Pattern
Before each task, load the relevant context into the AI agent:

```
System context:
  - Project constitution (principles and constraints)
  - Relevant sections of spec.md
  - Relevant sections of plan.md
  - Current task description
  - Existing code files the task modifies

Agent instruction:
  "Implement Task 3 per the plan. Validate against the acceptance
   criteria listed in the task. Follow the constitution's coding standards."
```

### Incremental Implementation
Don't implement everything at once. Follow the task sequence:

```
✅  Recommended: Implement Task 1 → Verify → Commit → Task 2 → Verify → ...
❌  Risky:       Implement all tasks at once → Hope everything works
```

### Handling Deviations
When implementation reveals the plan needs changes:

```
1. Document the deviation: "Task 4 revealed that X won't work because Y"
2. Propose a change: "Suggest changing plan §3 to use Z instead"
3. Update the plan (and spec if requirements are affected)
4. Continue implementation with the updated context
```

---

## Validation at Every Step

### Per-Task Validation
```
For Task N:
  □ Code compiles/runs without errors
  □ Unit tests written and passing
  □ Task acceptance criteria met
  □ No regressions in existing tests
```

### Per-Feature Validation
```
After all tasks:
  □ All acceptance criteria from spec.md pass
  □ Edge cases from spec.md are handled
  □ Non-functional requirements verified
  □ Integration tests passing
  □ E2E flow works as described in user stories
```

---

## Commit Strategy

### Commit Per Task
Each task gets its own commit with a structured message:

```
feat(notifications): create notification model and migration

Implements Task 2 of notification system spec.
- Creates Notification model with TypeScript validation
- Adds Prisma migration for notifications table
- Includes unit tests for model validation

Spec: .specify/specs/notifications/spec.md §2.1
Plan: .specify/specs/notifications/plan.md §3.1
```

### Benefits
- Git history mirrors the task structure
- Easy to bisect and find where specs were implemented
- Spec references in commits enable traceability

---

## CLI Usage

```bash
# Start implementation of the current spec
speckit implement

# In AI chat interface
/implement

# The AI agent will:
# 1. Read tasks.md
# 2. Implement each task sequentially
# 3. Run tests after each task
# 4. Report progress
```

---

## Working with AI Coding Agents

### GitHub Copilot Coding Agent
```markdown
# In your PR or issue:
"Implement the notification system per the spec in
.specify/specs/notifications/spec.md.
Follow the plan in plan.md and execute tasks in tasks.md order."
```

### Claude Code
```bash
claude "Implement Task 3 from .specify/specs/notifications/tasks.md.
Read plan.md for architecture context. Validate against spec.md acceptance criteria."
```

### Cursor / Windsurf
Load spec.md, plan.md, and tasks.md as context files. Reference the current task in your prompt.

---

## Common Implementation Pitfalls

### 1. Ignoring the Spec During Implementation
```
❌  AI generates code that "works" but doesn't match spec behavior
✅  After each task, explicitly check each acceptance criterion
```

### 2. Skipping Tests
```
❌  "I'll add tests later after all tasks are done"
✅  Each task includes its own tests — validated before moving on
```

### 3. Not Updating Specs on Deviation
```
❌  Code diverges from spec, spec becomes stale
✅  When code must differ from spec, update spec first, then code
```

### 4. Context Overload
```
❌  Loading the entire spec + plan + all tasks into one AI session
✅  Load only the current task + relevant spec sections + plan sections
```

---

## Quality Gate: Implementation Review

Verify all tasks complete, acceptance criteria pass, tests green, and spec updated with deviations. See [Quality Gates](quality-gates.md) for the full validation checklist for this phase.
