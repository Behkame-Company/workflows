# Instruction Hierarchy

> Understanding priority layers: system > developer > user — and why it matters.

---

## The Three Layers

```
Priority HIGH ──────────────────────────── Priority LOW

┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│   SYSTEM      │  │  DEVELOPER   │  │    USER      │
│              │  │              │  │              │
│ Set by the   │  │ Set by the   │  │ Set by the   │
│ platform     │  │ team/repo    │  │ individual   │
│              │  │              │  │              │
│ Tool safety  │  │ Project      │  │ Current task │
│ boundaries   │  │ conventions  │  │ request      │
└──────────────┘  └──────────────┘  └──────────────┘
```

---

## How Layers Map to Development Tools

| Layer | Copilot | Claude Code | Cursor | API |
|-------|---------|-------------|--------|-----|
| System | GitHub's built-in prompt | Anthropic's system | Cursor's base | `system` message |
| Developer | copilot-instructions.md | CLAUDE.md | .cursorrules | Custom system text |
| User | Chat messages, comments | Conversation messages | Chat | `user` messages |

---

## Priority Rules

### 1. System Layer Wins on Safety
The platform's system prompt sets hard boundaries (no harmful content, no credential leaking). These cannot be overridden by developer or user instructions.

### 2. Developer Layer Wins on Conventions
Your copilot-instructions.md or CLAUDE.md sets project-level rules. These should override user preferences when there's a conflict.

```markdown
# copilot-instructions.md (developer layer)
Always use Zod for input validation. 
Do not use manual type checking with typeof.
```

Even if a user asks "validate this input with typeof checks," the developer layer should take precedence.

### 3. User Layer Wins on Task Specifics
The current task request has the most up-to-date context:

```
User: "Generate the createUser function, but skip Zod validation 
for this prototype — we'll add it later."
```

This is a legitimate override of the developer convention for a specific reason.

---

## Designing for the Hierarchy

### In copilot-instructions.md (Developer Layer)

```markdown
# Hard Rules (never override)
- All API endpoints MUST validate input with Zod
- Never commit secrets or API keys
- All database queries go through the repository layer

# Soft Guidelines (may be overridden per-task)
- Prefer functional components (but class components OK for error boundaries)
- Target 80% test coverage (skip for prototypes if stated)
```

### When Layers Conflict

| Scenario | Winner | Why |
|----------|--------|-----|
| User asks to skip validation | Developer layer | Security concern |
| User asks for different test framework | User layer | Preference, not safety |
| User asks to ignore TypeScript errors | Developer layer | Code quality rule |
| User asks for a prototype without tests | User layer (if explicitly stated) | Temporary, acknowledged trade-off |

---

## Hierarchy in Prompt Files

```yaml
# .github/prompts/api-endpoint.prompt.md
---
description: "Generate a new API endpoint"
mode: "agent"
---

# System context (from copilot-instructions.md — automatic)
# Developer context (from this prompt file)
# User context (from the variables they provide)

Create an API endpoint for {{resource}}.
Follow our standard patterns. Override any default with 
specific instructions from the user.
```

---

## Next Steps

- 🔗 [Persona Engineering](persona-engineering.md) — Creating effective AI personas
- 🔗 [Constraint Specification](constraint-specification.md) — Defining boundaries
