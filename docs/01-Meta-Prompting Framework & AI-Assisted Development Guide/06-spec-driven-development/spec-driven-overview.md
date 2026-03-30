# Spec-Driven Development Overview

> A brief introduction to the specification-first approach for AI code generation.

---

## What Is Spec-Driven Development?

Spec-Driven Development (SDD) **flips the traditional workflow**: instead of writing code first and documenting later, you write comprehensive specifications first and let AI generate the implementation.

```
Traditional:    Idea → Code → Tests → Documentation
Spec-Driven:    Idea → Specification → AI Implementation → Validation
```

The specification becomes the **source of truth** — code is a direct expression of the spec, not the other way around.

---

## Core Philosophy

| Principle | Description |
|-----------|-------------|
| **Intent over implementation** | Define *what* you want before *how* to build it |
| **Rich specifications** | Detailed specs with edge cases, constraints, and acceptance criteria |
| **Multi-step refinement** | Specify → Clarify → Plan → Implement (not one-shot prompting) |
| **Specification as artifact** | Specs are version-controlled and maintained alongside code |

---

## The 6-Step Process (GitHub Spec Kit)

GitHub's [Spec Kit](https://github.github.com/spec-kit/) formalizes SDD into a structured workflow:

### Step 1: Constitution
Define the project's core principles and constraints.

> "This project follows Library-First design. All features must be standalone libraries first. We use TDD strictly."

### Step 2: Specify
Describe *what* you want to build — focus on the **what** and **why**, not the tech stack.

> "Build a photo album organizer. Albums are grouped by date, reorderable via drag-and-drop, and photos display in a tile layout."

### Step 3: Clarify
Identify and resolve ambiguities through structured questions.

> "What happens when a photo is in multiple albums? How large can albums get?"

### Step 4: Plan
Translate the specification into a technical implementation plan with architecture choices.

> "Use Next.js with Tailwind CSS. Store metadata in SQLite. Use @dnd-kit for drag-and-drop."

### Step 5: Tasks
Break the plan into discrete, implementable tasks.

### Step 6: Implement
Execute the plan with AI assistance, validating against the spec at each step.

---

## When to Use SDD

| Use SDD When... | Stick With Meta-Prompting When... |
|-----------------|-----------------------------------|
| Starting a new project from scratch | Adding features to an existing codebase |
| Complex, multi-component features | Small bug fixes or incremental changes |
| Need formal sign-off before building | Rapid prototyping and exploration |
| Compliance/audit trail required | Team is already iterating quickly |
| Multiple stakeholders need alignment | Solo developer workflow |

---

## How SDD Complements Meta-Prompting

SDD and meta-prompting are **not competing approaches** — they work at different levels:

```
SDD (Spec-Driven Development)
├── Governs: What to build, acceptance criteria, project lifecycle
└── Artifacts: Specifications, plans, task breakdowns

Meta-Prompting Framework
├── Governs: How AI operates in your codebase
└── Artifacts: Instruction files, memory bank, prompt templates
```

You can use SDD to define *what* to build and meta-prompting to ensure the AI builds it *correctly* according to your team's standards.

### Integration Example

1. **SDD** produces a specification for "User Authentication"
2. The spec becomes input for a **prompt file** (`implement-auth.prompt.md`)
3. **Path-specific instructions** ensure auth code follows security conventions
4. **Memory bank** tracks the implementation progress across sessions
5. **copilot-instructions.md** ensures all generated code meets team standards

---

## Spec Kit Installation (Quick Reference)

```bash
# Initialize a new project with Spec Kit
uvx --from git+https://github.com/github/spec-kit.git specify init my-project

# For Copilot-focused projects
uvx --from git+https://github.com/github/spec-kit.git specify init my-project --ai copilot

# For Claude Code projects
uvx --from git+https://github.com/github/spec-kit.git specify init my-project --ai claude
```

Requires: Python 3.11+, Git, [uv](https://docs.astral.sh/uv/) package manager.

---

## Further Reading

- [Spec Kit Documentation](https://github.github.com/spec-kit/)
- [Spec Kit Quick Start](https://github.github.com/spec-kit/quickstart.html)
- [SDD Methodology (Full)](https://github.com/github/spec-kit/blob/main/spec-driven.md)
- [Martin Fowler: Understanding SDD Tools](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html)
