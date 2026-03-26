# SDD + Meta-Prompting

> Combining Spec-Driven Development with instruction frameworks for maximum AI effectiveness.

---

## Two Systems, One Workflow

SDD and meta-prompting operate at different levels:

```
Meta-Prompting Framework (HOW the AI operates)
├── copilot-instructions.md  → Coding standards, patterns
├── .instructions.md         → Path-specific rules
├── AGENTS.md               → Cross-tool compatibility
├── Memory Bank             → Persistent context
└── Prompt Templates        → Reusable workflows

Spec-Driven Development (WHAT the AI builds)
├── constitution.md         → Project principles
├── spec.md                → Feature requirements
├── plan.md                → Technical architecture
└── tasks.md               → Work breakdown
```

**SDD defines the destination. Meta-prompting defines how the AI travels there.**

---

## Integration Points

### 1. Constitution ↔ copilot-instructions.md

The SDD constitution and Copilot instructions overlap. Manage the relationship:

```markdown
# Constitution (SDD - .specify/constitution.md)
## Technology Stack
- React 19, TypeScript 5.6, Fastify 5.x
## Quality Standards
- Test coverage >80%, no `any` types

# Copilot Instructions (Meta-Prompting - .github/copilot-instructions.md)
## When generating code:
- Use React functional components with TypeScript
- Use strict TypeScript (no `any` types)
- Always include JSDoc comments
- Follow patterns in src/utils/ for utility functions
```

**Rule**: Constitution defines *what* standards exist. Copilot instructions define *how to apply* them in code generation.

### 2. Spec → Prompt File

Convert spec acceptance criteria into executable prompt files:

```markdown
---
description: "Implement notification system from spec"
mode: agent
tools: ['terminal', 'editFiles']
---

# Context
Spec: .specify/specs/notifications/spec.md
Plan: .specify/specs/notifications/plan.md

# Task
Implement the next incomplete task from tasks.md.

# Constraints
- Follow coding standards in .github/copilot-instructions.md
- Validate each criterion from the spec before marking complete
- Commit after each task with spec reference
```

### 3. Memory Bank ↔ Spec History

The memory bank tracks implementation progress across sessions:

```markdown
# memory-bank/active-context.md

## Current Work
- Feature: Notification System
- Spec: .specify/specs/notifications/spec.md v1.2
- Progress: Tasks 1-4 complete, starting Task 5
- Deviations: Changed from WebSocket to SSE (spec updated in v1.2)
- Open Issues: Performance testing pending for AC-3.1
```

### 4. Path Instructions ↔ Spec Implementation Zones

Use path-specific instructions to enforce spec requirements in code:

```markdown
# src/features/notifications/.instructions.md
applyTo: "**/*.ts"

## Notification Module Standards (from spec v1.2)
- All notification types must implement the Notification interface
- Rate limiting: 100 notifications/hour/user (spec §3.2)
- All public methods must validate auth context
- Notification content sanitized against XSS (spec §4.1)
```

---

## The Combined Workflow

```
Step 1: Write spec (SDD)
Step 2: Review spec against constitution (SDD)
Step 3: Create plan with AI guided by copilot-instructions.md (Both)
Step 4: Generate tasks (SDD)
Step 5: Create prompt file for implementation (Meta-Prompting)
Step 6: Implement with path-specific instructions (Meta-Prompting)
Step 7: Validate against spec acceptance criteria (SDD)
Step 8: Update memory bank with progress (Meta-Prompting)
Step 9: Commit with spec reference (Both)
```

---

## File Hierarchy

```
project/
├── .github/
│   └── copilot-instructions.md     # How AI writes code
├── .specify/
│   ├── constitution.md              # Project standards
│   └── specs/
│       └── feature/
│           ├── spec.md              # What to build
│           ├── plan.md              # How to build it
│           └── tasks.md             # Work units
├── .prompts/
│   └── implement-feature.prompt.md  # Reusable implementation prompt
├── AGENTS.md                        # Cross-tool instructions
├── memory-bank/
│   └── active-context.md            # Current state
└── src/
    └── features/
        └── notifications/
            └── .instructions.md     # Module-level AI rules
```

---

## When to Use Which

| Situation | Use SDD | Use Meta-Prompting | Use Both |
|-----------|---------|-------------------|----------|
| New feature from scratch | ✅ | | ✅ |
| Bug fix | | ✅ | |
| Refactoring | | ✅ | |
| Complex feature implementation | | | ✅ |
| Setting up project standards | ✅ (constitution) | ✅ (instructions) | ✅ |
| Code review with AI | | ✅ | |
| Feature planning | ✅ | | |
| Daily coding tasks | | ✅ | |

---

*Next: [Enterprise SDD →](enterprise-sdd.md)*
