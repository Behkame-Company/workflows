# What Is Meta-Prompting?

> Teaching AI *how to think about your project*, not just *what to do*.

---

## Definition

**Meta-prompting** is the practice of creating higher-order instructions that guide how AI agents plan, reason, and execute tasks — rather than giving them individual task-level prompts. It's the difference between:

- **Prompting**: "Add a login form to the homepage"
- **Meta-prompting**: "When building UI features, always follow our component library, write tests first, and validate against the design system before committing"

Meta-prompts act as **persistent policies** that shape every interaction, creating consistent, predictable AI behavior across your entire team.

---

## Why It Matters for Teams

### The Problem Without Meta-Prompting

Without structured instructions, every AI interaction starts from zero:

- Developer A gets AI to use `async/await` — Developer B's session uses `.then()` chains
- The AI generates code without tests because nobody told it to
- Every PR review catches the same style violations that a meta-prompt could have prevented
- Context is lost between sessions; the AI forgets your architecture decisions

### The Solution

A meta-prompting framework provides:

| Benefit | How |
|---------|-----|
| **Consistency** | All team members get AI that follows the same conventions |
| **Quality** | AI validates its own work against your standards |
| **Efficiency** | No re-explaining project context every session |
| **Safety** | Explicit boundaries prevent dangerous operations |
| **Onboarding** | New team members (human or AI) ramp up instantly |

---

## Core Concepts

### 1. Instruction Hierarchy

Meta-prompts exist at multiple levels, from broadest to most specific:

```
Organization-wide instructions    (least specific)
    └── Repository instructions
        └── Path-specific instructions
            └── Session/task prompts   (most specific)
```

More specific instructions override or augment broader ones.

### 2. The Conductor-Expert Pattern

The most effective meta-prompting architecture separates **planning** from **execution**:

- **Conductor** (meta-prompt): Defines *how* to approach problems, decomposes tasks, assigns roles
- **Experts** (task-level): Execute specific sub-tasks with focused context

This prevents a single overloaded prompt and keeps each AI interaction focused and efficient.

### 3. Intent Over Implementation

Good meta-prompts define the **what** and **why**, letting the AI determine the **how**:

```markdown
# ❌ Too prescriptive (fragile)
Always use `const router = express.Router()` on line 1 of route files.

# ✅ Intent-driven (robust)
Route files should use the Express router pattern. Export a configured
router as the default export. Include input validation middleware.
```

### 4. Feedback Loops

Effective meta-prompting includes self-correction mechanisms:

1. **Before acting**: AI checks instructions for relevant constraints
2. **During execution**: AI validates against coding standards and tests
3. **After completion**: AI verifies the output meets acceptance criteria

---

## Meta-Prompting vs. Related Concepts

| Concept | Focus | Scope |
|---------|-------|-------|
| **Prompt Engineering** | Crafting individual prompts for one-off tasks | Single interaction |
| **Meta-Prompting** | Creating policies that govern all AI interactions | Persistent, cross-session |
| **Context Engineering** | Managing the full information environment for AI | Includes memory, RAG, tools |
| **Spec-Driven Development** | Writing specifications that generate code | Project lifecycle |

Meta-prompting is a **subset of context engineering** — it specifically focuses on the instruction layer, while context engineering encompasses memory, retrieval, tool schemas, and orchestration state.

---

## Practical Entry Points

For teams using **GitHub Copilot**, meta-prompting manifests through:

1. **`copilot-instructions.md`** — Repository-wide behavioral rules → 🔗 [Guide](../02-copilot-instructions/copilot-instructions-guide.md)
2. **`.instructions.md` files** — Path-scoped rules for specific code areas → 🔗 [Guide](../02-copilot-instructions/path-specific-instructions.md)
3. **`.prompt.md` files** — Reusable task-level prompt templates → 🔗 [Guide](../02-copilot-instructions/prompt-files.md)
4. **`AGENTS.md`** — Cross-tool agent instructions → 🔗 [Guide](../03-agents-md/agents-md-guide.md)
5. **Memory Bank** — Persistent project knowledge across sessions → 🔗 [Guide](../04-memory-bank/memory-bank-for-teams.md)

---

## Key Takeaways

- Meta-prompting is about **persistent policies**, not one-off prompts
- It creates **team-wide consistency** in AI-assisted development
- The instruction hierarchy goes from organization → repo → path → session
- Always define **intent and constraints**, not step-by-step scripts
- Include **feedback loops** so the AI validates its own output
