# Framework Architecture

> How to structure a meta-prompting framework using the Conductor-Expert pattern.

---

## Overview

A meta-prompting framework is a **layered system** of instruction files, memory, and workflows that governs how AI agents operate in your codebase. It's not a single file — it's an architecture.

```
┌───────────────────────────────────────────┐
│            Orchestration Layer             │
│   (How tasks are planned and decomposed)  │
├───────────────────────────────────────────┤
│            Instruction Layer              │
│   (Persistent rules and conventions)      │
├───────────────────────────────────────────┤
│             Memory Layer                  │
│   (Context that persists across sessions) │
├───────────────────────────────────────────┤
│            Execution Layer                │
│   (Task-level prompts and workflows)      │
├───────────────────────────────────────────┤
│           Verification Layer              │
│   (Self-validation and feedback loops)    │
└───────────────────────────────────────────┘
```

---

## The Conductor-Expert Pattern

The most effective architecture for AI-assisted development separates **planning** from **execution**:

### Conductor (Meta-Level)

The conductor doesn't write code — it decides *what to do* and *how to validate it*:

- Receives a high-level request ("Add user authentication")
- Decomposes it into discrete, testable tasks
- Assigns each task appropriate context (path-specific instructions, reference files)
- Defines acceptance criteria for each task
- Orchestrates the order of execution

In practice, the conductor is your **instruction files + prompt files + memory bank** working together.

### Experts (Task-Level)

Experts execute specific sub-tasks with focused context:

- Work on one bounded piece (e.g., "Create the JWT middleware")
- Receive only the relevant instructions for their domain
- Self-validate against provided criteria
- Report completion status

In practice, experts are **individual Copilot Chat sessions or Coding Agent tasks** that operate within the boundaries set by your instruction files.

### Flow

```
High-Level Request
       │
       ▼
┌─────────────────┐
│   CONDUCTOR      │  Reads: copilot-instructions.md, memory-bank/activeContext.md
│   (Planning)     │  Produces: Task breakdown, acceptance criteria
└────────┬────────┘
         │
    ┌────┼────┬────────┐
    ▼    ▼    ▼        ▼
┌──────┐┌──────┐┌──────┐┌──────┐
│Expert││Expert││Expert││Expert│  Each reads: path-specific .instructions.md
│  #1  ││  #2  ││  #3  ││  #4  │  Produces: Code changes, tests
└──┬───┘└──┬───┘└──┬───┘└──┬───┘
   │       │       │       │
   ▼       ▼       ▼       ▼
┌─────────────────────────────┐
│     VERIFICATION LAYER      │  Runs: lint, test, build, review
│    (Self-Validation)        │  Updates: memory-bank/activeContext.md
└─────────────────────────────┘
```

---

## Layer Details

### 1. Orchestration Layer

**Purpose**: Define *how* the AI approaches complex tasks.

**Implementation** in Copilot:

```markdown
# .github/prompts/plan-feature.prompt.md

# Feature Planning

Before implementing, create a plan:

1. **Scope**: List all files that will change
2. **Dependencies**: Identify what must be built first
3. **Risks**: Note anything that might break existing features
4. **Tests**: Define what tests will prove the feature works
5. **Order**: Sequence the implementation steps

Review: [System Patterns](../../memory-bank/systemPatterns.md)
Review: [Active Context](../../memory-bank/activeContext.md)

Do not start coding until the plan is reviewed.
```

### 2. Instruction Layer

**Purpose**: Persistent rules that apply automatically to all interactions.

**Implementation**:

| File | Role in Framework |
|------|-------------------|
| `copilot-instructions.md` | Global coding standards, build commands |
| `AGENTS.md` | Cross-tool universal rules |
| `frontend.instructions.md` | UI/component-specific conventions |
| `api.instructions.md` | Backend API conventions |
| `testing.instructions.md` | Test patterns and requirements |

### 3. Memory Layer

**Purpose**: Maintain continuity across sessions and context resets.

**Implementation**:

| File | Role in Framework |
|------|-------------------|
| `memory-bank/projectbrief.md` | Stable project scope |
| `memory-bank/systemPatterns.md` | Architecture decisions |
| `memory-bank/activeContext.md` | Current work state |
| `memory-bank/progress.md` | Milestone tracking |

### 4. Execution Layer

**Purpose**: Task-level prompts that guide specific implementation work.

**Implementation**:

| File | Role in Framework |
|------|-------------------|
| `add-feature.prompt.md` | Standard feature workflow |
| `fix-bug.prompt.md` | Bug fix procedure |
| `migrate-db.prompt.md` | Database change workflow |
| `code-review.prompt.md` | Review checklist |

### 5. Verification Layer

**Purpose**: Ensure the AI validates its own output.

**Implementation**: Embedded in instruction and prompt files:

```markdown
## Validation (in copilot-instructions.md)
After any code change:
1. `npm run lint` — must pass with zero errors
2. `npm run test` — all tests must pass
3. `npm run build` — must compile successfully
4. If new feature: demo that it works as specified
5. Update memory-bank/activeContext.md with completion status
```

---

## Modular Design Principles

### Separation of Concerns

Each file should have **one clear purpose**:

```
❌ copilot-instructions.md with 500 lines covering everything
✅ copilot-instructions.md (global rules) + 4 domain-specific .instructions.md files
```

### Single Responsibility per Layer

```
❌ Instructions that also try to track progress
✅ Instructions define rules → Memory bank tracks progress
```

### Information Hiding

Each layer only exposes what the next layer needs:

```
Orchestration → "Build the auth module following our patterns"
Instruction   → "Auth modules use JWT, live in src/auth/, need Zod validation"
Memory        → "Auth is planned for Sprint 4, no work started yet"
Execution     → The AI implements based on all the above
```

---

## Framework Scaling

### Small Project (1–3 developers)

```
.github/
├── copilot-instructions.md    # All rules in one file
└── prompts/
    └── add-feature.prompt.md  # One standard workflow
AGENTS.md                      # For cross-tool compat
```

### Medium Project (4–10 developers)

```
.github/
├── copilot-instructions.md
├── instructions/
│   ├── frontend.instructions.md
│   ├── api.instructions.md
│   └── testing.instructions.md
└── prompts/
    ├── add-feature.prompt.md
    ├── fix-bug.prompt.md
    └── code-review.prompt.md
AGENTS.md
memory-bank/
├── projectbrief.md
├── systemPatterns.md
├── activeContext.md
└── progress.md
```

### Large Project / Monorepo (10+ developers)

```
.github/
├── copilot-instructions.md        # Organization-level rules
├── instructions/
│   ├── frontend.instructions.md
│   ├── api.instructions.md
│   ├── database.instructions.md
│   ├── infrastructure.instructions.md
│   └── security.instructions.md
└── prompts/
    ├── add-feature.prompt.md
    ├── fix-bug.prompt.md
    ├── code-review.prompt.md
    ├── migrate-db.prompt.md
    └── deploy.prompt.md
AGENTS.md
memory-bank/
├── projectbrief.md
├── productContext.md
├── systemPatterns.md
├── techContext.md
├── activeContext.md
├── progress.md
└── domains/
    ├── auth.md
    ├── billing.md
    └── reporting.md
packages/
├── frontend/
│   └── AGENTS.md                  # Frontend-specific overrides
└── backend/
    └── AGENTS.md                  # Backend-specific overrides
```

---

*Next: [Building Your Framework](building-your-framework.md) →*
