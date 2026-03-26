# GitHub Spec Kit

> The open-source toolkit for Spec-Driven Development.

---

## Overview

[GitHub Spec Kit](https://github.com/github/spec-kit) is an MIT-licensed open-source toolkit that provides templates, CLI commands, and workflow structure for Spec-Driven Development.

---

## Installation

### Prerequisites
- Python 3.11+
- Git
- [uv](https://docs.astral.sh/uv/) package manager

### Install

```bash
# Install via uvx (recommended)
uvx --from git+https://github.com/github/spec-kit.git speckit

# Or clone and install locally
git clone https://github.com/github/spec-kit.git
cd spec-kit
pip install -e .
```

### Initialize a Project

```bash
# Basic initialization
speckit init my-project

# With AI tool configuration
speckit init my-project --ai copilot    # GitHub Copilot
speckit init my-project --ai claude     # Claude Code
speckit init my-project --ai cursor     # Cursor
speckit init my-project --ai gemini     # Gemini
```

---

## Directory Structure

After `speckit init`, your project gets:

```
.specify/
├── constitution.md          # Project principles and constraints
├── prompts/
│   ├── specify.md           # Prompt template for Specify phase
│   ├── plan.md              # Prompt template for Plan phase
│   ├── tasks.md             # Prompt template for Tasks phase
│   └── implement.md         # Prompt template for Implement phase
├── specs/                   # Your feature specifications go here
│   └── example/
│       ├── spec.md
│       ├── plan.md
│       └── tasks.md
└── templates/
    ├── spec-template.md     # Template for new specs
    └── plan-template.md     # Template for new plans
```

---

## CLI Commands

### `speckit constitution`
Create or edit the project constitution:
```bash
speckit constitution
# Opens interactive constitution creation
# Defines project principles, tech stack, quality standards
```

### `speckit specify`
Create a new feature specification:
```bash
speckit specify
# Interactive: asks clarifying questions
# Outputs: .specify/specs/<feature>/spec.md
```

### `speckit plan`
Generate a technical plan from an approved spec:
```bash
speckit plan
# Reads: spec.md
# Outputs: plan.md (architecture, data model, API design)
```

### `speckit tasks`
Break a plan into implementation tasks:
```bash
speckit tasks
# Reads: plan.md
# Outputs: tasks.md (ordered checklist with dependencies)
```

### `speckit implement`
Execute implementation guided by tasks:
```bash
speckit implement
# Reads: tasks.md
# Implements each task with AI assistance
# Validates against spec.md acceptance criteria
```

---

## Using Spec Kit with Chat Interfaces

In AI chat interfaces (Copilot Chat, Claude, Cursor), use slash commands:

| Command | Phase |
|---------|-------|
| `/specify` | Create or refine a specification |
| `/plan` | Generate implementation plan |
| `/tasks` | Break plan into tasks |
| `/implement` | Start implementing |

The prompts in `.specify/prompts/` customize how each command behaves.

---

## Customizing Templates

### Spec Template
Edit `.specify/templates/spec-template.md`:

```markdown
# Feature: {{feature_name}}

## Summary
{{description}}

## User Stories
- As a {{user_type}}, I want {{goal}}, so that {{benefit}}

## Acceptance Criteria
- [ ] {{criterion}}

## Edge Cases
| Scenario | Expected Behavior |
|----------|------------------|

## Non-Functional Requirements

## Out of Scope
```

### Prompt Customization
Edit `.specify/prompts/specify.md` to change how the AI guides specification:

```markdown
You are a specification assistant. Help the user write a detailed spec.

Rules:
- Ask clarifying questions before writing
- Always include acceptance criteria in Given/When/Then format
- Include at least 3 edge cases
- Define what's out of scope
- Reference the project constitution for standards
```

---

## Integration with Existing Workflows

### With copilot-instructions.md
```markdown
# .github/copilot-instructions.md
When implementing features, always check .specify/specs/ for a specification.
If a spec exists, validate your implementation against its acceptance criteria.
```

### With AGENTS.md
```markdown
# AGENTS.md
## Spec-Driven Workflow
- Check .specify/specs/ for feature specifications before implementing
- Reference spec acceptance criteria in commit messages
- Update spec if implementation deviates
```

### With CI/CD
```yaml
- name: Validate specs
  run: speckit validate
- name: Check spec coverage
  run: speckit coverage
```

---

## Strengths and Limitations

### Strengths
- ✅ Open source (MIT license)
- ✅ Tool-agnostic (works with any AI assistant)
- ✅ Lightweight (Markdown files, no database)
- ✅ Customizable templates and prompts
- ✅ Git-friendly (all files version-controlled)
- ✅ Active development by GitHub

### Limitations
- ⚠️ CLI requires Python 3.11+
- ⚠️ No built-in drift detection
- ⚠️ No GUI — CLI and Markdown only
- ⚠️ Limited automation (manual quality gates)
- ⚠️ Templates may need significant customization

---

*Next: [AWS Kiro →](aws-kiro.md)*
