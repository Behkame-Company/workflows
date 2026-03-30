# From Prompt Engineering to Context Engineering

> The paradigm shift from crafting prompts to curating entire context states.

---

## The Evolution

```
2022-2023: PROMPT ENGINEERING
  Focus: Writing the right words
  Question: "How do I phrase this prompt?"
  Scope: Single instruction

2024: PROMPT + CONTEXT
  Focus: Instructions + relevant data
  Question: "What files should I attach?"
  Scope: Prompt + supporting context

2025-2026: CONTEXT ENGINEERING
  Focus: Curating the entire token state
  Question: "What configuration of context maximizes desired behavior?"
  Scope: Everything the model can see
```

---

## What Changed

| Aspect | Prompt Engineering | Context Engineering |
|--------|-------------------|-------------------|
| **Focus** | Wording of instructions | Entire token state |
| **Scope** | System prompt | System + tools + history + data |
| **Timing** | Before inference | Continuous (every turn) |
| **Nature** | Static | Dynamic, iterative |
| **Goal** | Get the right answer | Maintain coherent agent behavior |
| **Key skill** | Writing clear instructions | Curating information flow |

---

## Context Engineering in Practice

Context engineering means managing **everything** the model sees at each inference step:

```
System Prompt         → Instructions, persona, rules
Tool Definitions      → Available capabilities
Tool Results          → Data retrieved by tools
Conversation History  → What's been said and done
Injected Context      → Files, docs, code references
Memory/Notes          → Persisted state from prior work
```

At each turn, you decide:
- What stays in context?
- What gets compacted or summarized?
- What gets retrieved on demand?
- What gets delegated to a sub-agent?

---

## The Core Principle

> **Find the smallest possible set of high-signal tokens that maximize the likelihood of the desired outcome.**

This principle is formalized as Minimal Viable Context — see [MVC](../09-best-practices/minimal-viable-context.md).

It means:
1. **Every token should earn its place** — If it doesn't help, it hurts
2. **Context is a finite resource with diminishing returns** — More isn't better
3. **Curation is continuous** — Not just at the start, but every turn
4. **The model's attention budget is real** — Treat it like a scarce resource

---

## Key Takeaways

1. **Context engineering > prompt engineering** — It's the bigger game
2. **Curate continuously** — Every turn, optimize what's in the window
3. **Smallest viable context** — Only what's needed, nothing more
4. **Dynamic, not static** — Context changes as the task evolves
5. **This is the meta-skill** — Master this to master AI-assisted development
