# AWS Kiro

> Amazon's spec-driven agentic IDE.

---

## Overview

[AWS Kiro](https://kiro.dev) is a full IDE built on Code OSS (VS Code's open-source core) that makes spec-driven development the native workflow. Unlike Spec Kit (which adds SDD to any editor), Kiro integrates SDD into the IDE itself.

---

## Core Concepts

### 1. Specs (Requirements)
Kiro generates structured requirements using **EARS** (Easy Approach to Requirements Syntax):

```
WHEN a user clicks Save,
THE SYSTEM SHALL store the data and show a success message.

WHILE the upload is in progress,
THE SYSTEM SHALL display a progress indicator.

IF the file exceeds 10MB,
THEN THE SYSTEM SHALL show an error message.
```

### 2. Design Docs
Kiro auto-generates technical design documents:
- Architecture diagrams
- Data flow diagrams
- TypeScript interfaces
- Database schemas
- API endpoint specifications

### 3. Implementation Tasks
Work is decomposed into ordered, reviewable tasks linked back to requirements and design.

---

## The Kiro Workflow

```
1. Prompt  → Describe what you want to build (natural language)
2. Specs   → Kiro generates requirements in EARS format
3. Design  → Kiro proposes architecture and data models
4. Tasks   → Kiro creates implementation checklist
5. Code    → Kiro implements tasks, validating against specs
```

### Artifacts Generated

| Artifact | File | Description |
|----------|------|-------------|
| Requirements | `requirements.md` | EARS-format user stories and criteria |
| Design | `design.md` | Architecture, schemas, interfaces |
| Tasks | `tasks.md` | Ordered implementation checklist |

All stored as Markdown in the repository.

---

## Steering Files

Kiro's equivalent of `copilot-instructions.md` — persistent context files in `.kiro/steering/`:

```
.kiro/
├── steering/
│   ├── coding-standards.md    # Code style and conventions
│   ├── tech-stack.md          # Technology choices
│   ├── naming-conventions.md  # Naming patterns
│   └── security.md            # Security requirements
```

**Key difference**: Steering files are always loaded — you don't need to re-explain your standards to the AI each session.

---

## Hooks (Automated Actions)

Hooks are event-triggered automations:

```
Triggers:
- File save → Run linter, update tests
- Commit → Check spec compliance
- New file → Apply boilerplate
- API change → Update OpenAPI docs
```

### Example Hook: Auto-Generate Tests
```markdown
# .kiro/hooks/auto-test.md
Trigger: On save of any file in src/
Action: Generate or update unit tests for the changed file
Rules: Follow existing test patterns, use Vitest
```

---

## Kiro vs Spec Kit

| Feature | GitHub Spec Kit | AWS Kiro |
|---------|----------------|----------|
| **Type** | CLI toolkit + templates | Full IDE |
| **Base** | Any editor | Code OSS (VS Code fork) |
| **Spec format** | Freeform Markdown | EARS-format structured |
| **Plan generation** | AI-assisted | AI-native |
| **Task tracking** | Manual Markdown | Integrated checklist |
| **Hooks/automation** | No | Yes (event-driven) |
| **Steering/context** | Via copilot-instructions.md | Native steering files |
| **Cost** | Free (open source) | Free preview, pricing TBA |
| **Model support** | Any AI model | Amazon Bedrock models |
| **VS Code extensions** | N/A (editor-agnostic) | Full VS Code extension support |
| **Maturity** | Production-ready | Preview/early access |

---

## When to Choose Kiro

### Choose Kiro If:
- You want SDD built into the IDE (zero setup)
- Your team uses AWS and Amazon Bedrock
- You want automated hooks for CI-like tasks
- You prefer structured EARS requirements over freeform
- You want an all-in-one solution

### Choose Spec Kit If:
- You want to use your preferred editor
- You need model-agnostic flexibility
- You want a lightweight, open-source solution
- Your team uses GitHub Copilot as primary AI tool
- You prefer freeform Markdown specifications

---

## Getting Started with Kiro

1. Download Kiro from [kiro.dev](https://kiro.dev)
2. Open your project
3. Create steering files in `.kiro/steering/`
4. Start a new feature: describe what you want to build
5. Review generated specs, design, and tasks
6. Approve and implement

---

*Next: [Other SDD Tools →](other-tools.md)*
