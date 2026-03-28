# System Prompt Design

> Principles of effective system prompts — the foundation of every AI interaction.

---

## What Is a System Prompt?

The system prompt is the first message in any AI conversation. It sets the identity, capabilities, constraints, and behavior of the AI for the entire session. In development tools, this maps to:

| Tool | System Prompt Location |
|------|----------------------|
| GitHub Copilot | copilot-instructions.md + tool-injected context |
| Claude Code | CLAUDE.md + project configuration |
| Cursor | .cursorrules |
| OpenAI API | `system` role message |
| Anthropic API | `system` parameter |

---

## The Right Altitude

Anthropic's core principle for system prompts: **Write at the right altitude.**

```
Too high (useless):
  "Be a good developer"

Too low (micromanaging):
  "When you see a function with more than 3 parameters, 
   create a params object with interface X..."

Right altitude:
  "Prefer small functions with clear names. Extract parameter 
   objects when functions exceed 3 parameters."
```

The right altitude gives direction without prescribing every step.

---

## System Prompt Structure

```markdown
# Identity and Role
[Who the AI is in this context]

# Core Rules (5-15 max)
[Non-negotiable project conventions]

# Project Context
[Stack, architecture, key patterns]

# Behavior Guidelines
[How to approach tasks, when to ask questions]

# Output Preferences
[Format, verbosity, style]
```

---

## Key Principles

### 1. Fewer Rules, Better Followed
```
❌ 50 rules covering every scenario
   → Model forgets half of them

✅ 10-15 critical rules
   → Model follows them consistently
```

### 2. Rules with Rationale
```
❌ "Always use early returns"

✅ "Use early returns for guard clauses — this reduces nesting 
    depth and makes the happy path easier to read"
```

Claude generalizes better when it understands **why**.

### 3. Positive Instructions
```
❌ "Don't use any types"
   "Don't write long functions"
   "Don't use console.log"

✅ "Use explicit TypeScript types for all parameters and returns"
   "Keep functions under 25 lines, extracting helpers as needed"
   "Use the logger service for all output"
```

### 4. Examples Over Descriptions
```
Error format: { error: string, code: string, details?: unknown }

Example:
{ "error": "User not found", "code": "USER_NOT_FOUND", "details": { "id": 42 } }
```

---

## Token Budget for System Prompts

| Section | Target | Notes |
|---------|--------|-------|
| Identity/Role | 50-100 tokens | One clear paragraph |
| Core Rules | 200-500 tokens | 10-15 rules with examples |
| Project Context | 100-300 tokens | Stack, structure, key files |
| Behavior | 100-200 tokens | Approach, tone, escalation |
| **Total** | **500-1,100 tokens** | ~2-4 pages of text |

> System prompts over 2,000 tokens often hurt more than they help — too many rules means none are followed reliably.

---

## Model-Specific Considerations

### Claude 4.x (Opus, Sonnet)
- Very responsive to system prompts — even subtle wording changes have effect
- Previous "CRITICAL: YOU MUST..." emphasis is now overkill — normal language works
- May overtrigger on aggressive tool-use instructions; dial back
- Extended thinking is influenced by system prompt complexity

### GPT-4 / GPT-4o
- Follows structured system prompts well
- Benefits from numbered rules
- May need stronger emphasis for constraint adherence

---

## Next Steps

- 🔗 [Instruction Hierarchy](instruction-hierarchy.md) — Priority between system/developer/user
- 🔗 [Persona Engineering](persona-engineering.md) — Deep dive into AI personas
