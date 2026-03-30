# Prompt Engineering vs Context Engineering

> Where prompt engineering fits in the broader picture of AI-assisted development.

---

## The Spectrum

```
Narrower Focus                                    Broader Focus
─────────────────────────────────────────────────────────────►
 Prompt         Prompt           Context          Agentic
 Crafting       Engineering      Engineering      Systems
 
 "What words    "Systematic      "Curating ALL    "Teaching AI
  work best?"   techniques for    tokens the       how to use
                 reliable         model sees"      tools, plan,
                 outputs"                          and reason"
```

---

## Definitions

### Prompt Engineering
Designing the **instruction text** you give to a model — the question, task description, examples, and constraints.

**Scope**: The text you write in a single prompt or system message.

### Context Engineering
Curating **all tokens** the model sees at each inference step — system prompt, conversation history, tool results, retrieved data, and the response itself.

**Scope**: Everything in the context window.

> "Context engineering is the art and science of filling the context window with exactly the right information for the next step." — Anthropic

---

## Why the Distinction Matters

| Aspect | Prompt Engineering | Context Engineering |
|--------|-------------------|-------------------|
| Focus | The question/instruction | Everything the model sees |
| Controls | Wording, structure, examples | History, tools, retrieval, compaction |
| Failure mode | Bad answer | Good answer to wrong question |
| Analogy | Writing good emails | Curating the entire briefing package |

### Example

**Prompt engineering problem**: "The model doesn't format output correctly"
→ Fix: Better format instructions, add examples

**Context engineering problem**: "The model uses old patterns instead of our new conventions"
→ Fix: Update system prompt, remove stale tool results, inject recent convention changes

---

## How They Work Together

```
Context Engineering (everything)
├── System Prompt ← prompt engineering
├── Conversation History ← context management
├── Tool Results ← retrieval engineering
├── User Message ← prompt engineering
└── Response Space ← token budgeting
```

Prompt engineering is a **subset** of context engineering. You need both:
1. Well-crafted prompts (clear instructions, good examples)
2. Well-managed context (right information at the right time)

---

## When to Focus on What

| Situation | Focus On |
|-----------|----------|
| Model gives wrong format | Prompt engineering (better format spec) |
| Model forgets earlier decisions | Context engineering (compaction, memory) |
| Model hallucinating file names | Context engineering (tool results, retrieval) |
| Model not following conventions | Both (better instructions + right context) |
| Model getting slower/worse over time | Context engineering (window management) |

---

## For This Documentation Set

This section (Phase 8) focuses on **prompt engineering patterns** — the techniques for crafting effective individual prompts.

For a deeper dive into context engineering, see [Context Window Management](../../07-Context%20Window%20Management/).

Also see:
- 🔗 [Copilot Instructions](../../Copilot%20instructions/) — Persistent system-level instructions
