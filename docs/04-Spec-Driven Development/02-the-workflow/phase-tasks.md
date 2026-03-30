# Phase 3: Tasks

> Breaking the plan into discrete, implementable units of work.

---

## Purpose

The Tasks phase transforms the plan into a **sequenced checklist** of concrete work items that an AI agent or developer can execute independently.

---

## Task Structure Template

```markdown
# Tasks: [Feature Name]

## Overview
Total: [N] tasks | Dependencies: [diagram] | Parallelizable: [groups]

## Task 1: [Create database migration]
- **Description**: Create Prisma migration adding notifications table
- **Files**: `prisma/migrations/xxx_add_notifications.sql`
- **Acceptance**: Migration runs successfully, table exists with correct schema
- **Dependencies**: None (starter task)
- **Estimated complexity**: Low

## Task 2: [Implement Notification model]
- **Description**: Create Notification TypeScript model with validation
- **Files**: `src/models/notification.ts`, `src/models/__tests__/notification.test.ts`
- **Acceptance**: Model validates all fields, unit tests pass
- **Dependencies**: Task 1

## Task 3: [Build notification API endpoints]
...
```

---

## Task Decomposition Principles

### The INVEST Criteria
Good tasks follow INVEST (adapted from user stories):

| Criterion | Meaning | Example |
|-----------|---------|---------|
| **I**ndependent | Minimal dependencies | Can be built without waiting |
| **N**egotiable | Details can be adjusted | Not locked to one approach |
| **V**aluable | Delivers a testable increment | Creates something verifiable |
| **E**stimable | Scope is clear enough to estimate | Not "figure out auth" |
| **S**mall | Completable in one session | Hours, not days |
| **T**estable | Has clear success criteria | "Test X passes" |

### Sizing Guide

| Too Large | Just Right | Too Small |
|-----------|-----------|-----------|
| "Build auth system" | "Create login API endpoint with JWT" | "Import bcrypt" |
| "Add dashboard" | "Create dashboard layout with sidebar navigation" | "Add CSS class" |
| "Set up database" | "Create User and Post tables with Prisma migration" | "Add column constraint" |

---

## Dependency Mapping

### Visual Dependency Graph
```
Task 1: DB Migration ─────────────────────────┐
Task 2: Model Layer  ──── depends on Task 1 ──┤
Task 3: Repository   ──── depends on Task 2 ──┤
Task 4: Service Logic ─── depends on Task 3 ──┤
Task 5: API Routes   ──── depends on Task 4 ──┤
Task 6: Frontend UI  ──── depends on Task 5 ──┘
Task 7: Unit Tests   ──── depends on Tasks 2-4 (parallel group)
Task 8: E2E Tests    ──── depends on Tasks 5-6
```

### Parallelization
Identify tasks that can run concurrently:

```
Wave 1 (parallel):  Task 1 (DB), Task 7a (test setup)
Wave 2 (parallel):  Task 2 (Model), Task 3 (Repo)
Wave 3 (sequential): Task 4 (Service) → Task 5 (API)
Wave 4 (parallel):  Task 6 (UI), Task 8 (E2E Tests)
```

---

## Task Metadata

Each task should include:

```markdown
## Task N: [Title]

### Required Fields
- **Description**: What to do (2-3 sentences)
- **Files**: New or modified files
- **Acceptance**: How to verify completion
- **Dependencies**: Tasks that must complete first

### Optional Fields
- **Complexity**: Low / Medium / High
- **Parallelizable with**: [Task N, Task M]
- **Notes**: Implementation hints or constraints
- **Spec reference**: Which spec requirement this fulfills
```

---

## Common Task Patterns

### Pattern: Database First
```
1. Create migration ──► 2. Create model ──► 3. Create repository
4. Seed test data (parallel with 3)
```

### Pattern: API Layer
```
1. Define types/interfaces ──► 2. Create service ──► 3. Create routes
4. Add middleware (parallel with 3) ──► 5. API tests
```

### Pattern: Frontend Feature
```
1. Create component ──► 2. Add state management ──► 3. Connect to API
4. Add loading/error states ──► 5. Add tests ──► 6. Responsive styling
```

### Pattern: Cross-Cutting Concern
```
1. Create utility/helper ──► 2. Integrate into existing services
3. Update existing tests ──► 4. Add new tests
```

---

## AI Task Generation Tips

When asking AI to generate tasks:

```
Prompt:  "Given this plan [paste plan.md], generate an ordered task list.
          Each task should:
          - Be completable in a single AI session
          - Have clear file paths for new/modified files
          - Include testable acceptance criteria
          - Note dependencies on other tasks
          Group parallelizable tasks together."
```

### Review the generated tasks for:
- [ ] Every plan item has corresponding tasks
- [ ] No task is larger than ~1 hour of work
- [ ] Dependencies form a valid DAG (no cycles)
- [ ] Acceptance criteria are concrete, not vague
- [ ] File paths match your project structure

---

## Quality Gate: Task Review

Verify completeness, granularity, dependencies, and test coverage of all tasks. See [Quality Gates](quality-gates.md) for the full validation checklist for this phase.
