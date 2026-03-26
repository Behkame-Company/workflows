# The 4-Phase Lifecycle

> Specify → Plan → Tasks → Implement — the complete SDD workflow explained.

---

## Overview

Spec-Driven Development follows a structured, gate-checked lifecycle with four core phases:

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────────┐
│ SPECIFY  │───►│   PLAN   │───►│  TASKS   │───►│  IMPLEMENT   │
│          │    │          │    │          │    │              │
│ What &   │    │ How &    │    │ Work     │    │ Code &       │
│ Why      │    │ Where    │    │ Units    │    │ Validate     │
└──────────┘    └──────────┘    └──────────┘    └──────────────┘
      │               │               │                │
      ▼               ▼               ▼                ▼
  [Quality Gate]  [Quality Gate]  [Quality Gate]   [Quality Gate]
  Human reviews   Human reviews   Human reviews    Code review +
  spec accuracy   plan coverage   task scope       acceptance test
```

---

## Phase 1: Specify

**Purpose**: Capture *what* to build and *why*.

**Inputs**: Feature request, user feedback, product roadmap
**Outputs**: `spec.md` — a structured specification

### What Goes in a Spec
- Feature description and user goals
- User stories with acceptance criteria
- Edge cases and error scenarios
- Non-functional requirements (performance, security, accessibility)
- Out-of-scope declaration
- Success metrics

### CLI Command
```bash
speckit specify          # Interactive spec creation
/specify                  # In AI chat interface
```

### Quality Gate
Before moving to Plan, verify:
- [ ] All stakeholders have reviewed the spec
- [ ] Acceptance criteria are testable
- [ ] Edge cases are documented
- [ ] Scope boundaries are clear

---

## Phase 2: Plan

**Purpose**: Determine *how* the spec will be implemented.

**Inputs**: Approved `spec.md`
**Outputs**: `plan.md` — technical architecture and approach

### What Goes in a Plan
- Architecture decisions (frameworks, patterns, technologies)
- Data model and database schema
- API contracts and endpoints
- Integration points with existing code
- Testing strategy
- Deployment considerations

### CLI Command
```bash
speckit plan              # Generate plan from spec
/plan                     # In AI chat interface
```

### Quality Gate
Before moving to Tasks, verify:
- [ ] Plan addresses all spec requirements
- [ ] Architecture choices are justified
- [ ] No spec requirement is left unplanned
- [ ] Technical risks are identified

---

## Phase 3: Tasks

**Purpose**: Break the plan into discrete, implementable work units.

**Inputs**: Approved `plan.md`
**Outputs**: `tasks.md` — checklist of implementation tasks

### What Goes in Tasks
- Ordered list of implementation tasks
- Dependencies between tasks
- Estimated complexity per task
- Testing requirements per task
- Parallelization opportunities

### CLI Command
```bash
speckit tasks             # Generate task breakdown
/tasks                    # In AI chat interface
```

### Task Decomposition Principles
```
Good Task:  "Create User model with email, hashed_password, created_at fields"
Bad Task:   "Build user system"  (too broad)
Bad Task:   "Add semicolon to line 47"  (too granular — that's a code change, not a task)
```

### Quality Gate
Before moving to Implement, verify:
- [ ] Every plan item maps to at least one task
- [ ] Tasks are independently implementable
- [ ] Task ordering respects dependencies
- [ ] Each task has clear completion criteria

---

## Phase 4: Implement

**Purpose**: Execute the tasks, generating code validated against the spec.

**Inputs**: Approved `tasks.md`
**Outputs**: Working code, tests, documentation

### Implementation Flow
```
For each task:
  1. Read task requirements
  2. Generate code (AI or manual)
  3. Validate against acceptance criteria
  4. Run tests
  5. Mark task complete
  6. Move to next task
```

### CLI Command
```bash
speckit implement         # AI-assisted implementation
/implement                # In AI chat interface
```

### Quality Gate
Before merging, verify:
- [ ] All tasks are completed
- [ ] All acceptance criteria pass
- [ ] Tests are written and green
- [ ] Code review completed
- [ ] Spec is updated if any deviations occurred

---

## The Constitution (Phase 0)

Before starting any specs, define your project's **constitution** — the invariant principles:

```markdown
## Project Constitution

### Design Principles
- Library-first: All features must work as standalone libraries
- Mobile-first: Responsive design as default
- Accessibility: WCAG 2.1 AA compliance required

### Technology Constraints
- Frontend: React 19 + TypeScript 5
- Backend: Node.js 22 + Fastify
- Database: PostgreSQL 16

### Quality Standards
- Test coverage: >80% for all new code
- No `any` types in TypeScript
- All API endpoints documented with OpenAPI
```

---

## Iteration and Feedback Loops

SDD is **not** a one-way waterfall. Feedback flows backward:

```
Specify ─────────────► Plan
   ▲                     │
   │     ┌───────────────┘
   │     ▼
   │   Tasks
   │     │
   │     ▼
   └── Implement ──── "Spec missed edge case X"
                      "Plan needs architecture change Y"
```

When implementation reveals new information:
1. **Update the spec** (not just the code)
2. Re-evaluate the plan if architecture is affected
3. Add tasks if scope expanded
4. Document the deviation and rationale

---

## Phase Duration Guidelines

| Project Size | Specify | Plan | Tasks | Implement |
|-------------|---------|------|-------|-----------|
| Small feature | 30 min | 15 min | 10 min | Hours |
| Medium feature | 2 hours | 1 hour | 30 min | Days |
| Large feature | 1 day | Half day | 2 hours | Weeks |
| System redesign | Days | Days | 1 day | Months |

---

*Next: [Phase 1: Specify →](phase-specify.md)*
