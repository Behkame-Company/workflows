# Integrating SDD with AI Agents

> Using Spec-Driven Development with GitHub Copilot, Claude Code, Cursor, and other AI tools.

---

## The Universal Pattern

Regardless of the AI tool, the SDD integration pattern is the same:

```
1. Load spec context into the AI agent
2. Reference specific requirements and acceptance criteria
3. Validate AI output against the spec
4. Update spec if deviations are necessary
```

---

## GitHub Copilot

### Copilot Chat (IDE)
```
@workspace Read the specification in .specify/specs/notifications/spec.md.
Based on the acceptance criteria, implement the notification service.
Follow the plan in plan.md for architecture decisions.
```

### Copilot Coding Agent (PR-based)
Create an issue or PR description that references specs:

```markdown
## Task: Implement Notification Service

Please implement the notification system as specified in:
- **Spec**: `.specify/specs/notifications/spec.md`
- **Plan**: `.specify/specs/notifications/plan.md`
- **Tasks**: `.specify/specs/notifications/tasks.md`

Implement tasks 1-5 in order. For each task:
1. Read the task requirements
2. Implement the code
3. Write tests
4. Validate against acceptance criteria in spec.md
```

### Integration with copilot-instructions.md
```markdown
# .github/copilot-instructions.md

## Spec-Driven Workflow
- Before implementing a feature, check .specify/specs/ for a specification
- Reference spec acceptance criteria when generating tests
- If no spec exists, ask whether one should be created first
- Include spec references in commit messages: "Spec: path §section"
```

### Integration with .prompt.md
```markdown
---
description: "Implement a feature from its specification"
mode: agent
tools: ['terminal', 'editFiles']
---

Read the specification at ${{ input.specPath }}.
Implement each acceptance criterion as follows:
1. Write a failing test for the criterion
2. Implement the code to pass the test
3. Verify against the spec language
4. Report which criteria are complete
```

---

## Claude Code

### CLAUDE.md Integration
```markdown
# CLAUDE.md

## Spec-Driven Development
This project uses Spec-Driven Development. Specifications are in .specify/specs/.
- Always read the relevant spec before implementing a feature
- Validate your work against spec acceptance criteria
- If you find a gap in the spec, note it but don't implement unspecified behavior
- Reference spec sections in your responses: "Per spec §2.3..."
```

### Claude CLI Usage
```bash
# Implement from spec
claude "Read .specify/specs/notifications/spec.md and implement Task 3 
from tasks.md. Validate against acceptance criteria AC-2.1 through AC-2.4."

# Validate implementation
claude "Read .specify/specs/notifications/spec.md. 
Check if the current implementation in src/services/notifications/ 
satisfies all acceptance criteria. Report any gaps."

# Draft a spec
claude "I want to add a search feature to our product catalog. 
Ask me clarifying questions, then write a spec following the template 
in .specify/templates/spec-template.md"
```

---

## Cursor

### .cursorrules Integration
```markdown
# .cursorrules

## Spec Awareness
- Check .specify/specs/ for specifications before implementing features
- Reference spec acceptance criteria in generated tests
- Follow the plan.md architecture for implementation decisions

## Spec-Driven Commands
When I say "implement spec [name]":
1. Read .specify/specs/[name]/spec.md
2. Read .specify/specs/[name]/plan.md
3. Read .specify/specs/[name]/tasks.md
4. Implement tasks sequentially, validating against spec
```

### Context Loading
Add spec files to Cursor context:
1. Open spec.md, plan.md, tasks.md
2. Add them to the chat context with @file
3. Prompt: "Implement Task 2 based on these specs"

---

## Multi-Agent SDD Workflows

### Agent Specialization

```
Spec Agent:     Drafts and refines specifications
Plan Agent:     Creates technical plans from specs
Task Agent:     Decomposes plans into tasks
Code Agent:     Implements tasks
Review Agent:   Validates implementation against spec
```

### Example: Copilot CLI + Spec Kit

```bash
# Phase 1: Specify (use Copilot CLI for drafting)
copilot "Help me write a spec for a notification system.
Use the template in .specify/templates/spec-template.md.
Ask me 5 clarifying questions first."

# Phase 2: Plan (AI-generated, human-reviewed)
speckit plan

# Phase 3: Tasks (AI-generated, human-reviewed)  
speckit tasks

# Phase 4: Implement (task by task)
copilot "Read tasks.md and implement Task 1.
Follow the plan in plan.md.
Validate against spec.md acceptance criteria."
```

---

## Best Practices for AI + SDD

### 1. Load Minimal Relevant Context
```
❌  Load entire spec + plan + all tasks + all code
✅  Load current task + relevant spec section + relevant plan section
```

### 2. Validate After Each Task
```
After implementation:
  "Check if Task 3 satisfies AC-2.1 through AC-2.3 from spec.md.
   Report pass/fail for each criterion."
```

### 3. Use Structured Commit Messages
```
feat(notifications): implement real-time delivery

Spec: .specify/specs/notifications/spec.md
Tasks: 1-3 of tasks.md
Criteria: AC-1.1 ✅, AC-1.2 ✅, AC-1.3 ✅
```

### 4. Keep Specs Accessible to Agents
Put specs where AI agents can find them:
```
✅  .specify/specs/ (in repo, discoverable)
❌  Google Docs (AI can't read it)
❌  Confluence (requires authentication)
❌  Slack messages (ephemeral, unsearchable)
```

---

*Next: [SDD + Meta-Prompting →](../07-advanced/sdd-plus-meta-prompting.md)*
