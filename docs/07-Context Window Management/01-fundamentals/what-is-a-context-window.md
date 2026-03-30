# What Is a Context Window?

> The finite "working memory" that determines everything your AI agent can see and reason about.

---

## Definition

A **context window** is the total number of tokens a language model can reference when generating a response. It includes:

- **System prompt** — Your instructions, rules, persona
- **Conversation history** — All previous messages in the session
- **Tool definitions** — Available tools and their schemas
- **Tool results** — Output from tool calls (files read, searches, etc.)
- **The response itself** — The model's output counts toward the window

```
┌─────────────────────────────────────────────────┐
│                 CONTEXT WINDOW                   │
│                                                  │
│  ┌──────────────────────────────────────────┐   │
│  │ System Prompt          (~2,000 tokens)    │   │
│  ├──────────────────────────────────────────┤   │
│  │ Tool Definitions       (~3,000 tokens)    │   │
│  ├──────────────────────────────────────────┤   │
│  │ Conversation History   (grows over time)  │   │
│  │  User: "..."                              │   │
│  │  Assistant: "..."                         │   │
│  │  User: "..."                              │   │
│  │  Tool Result: { ... }                     │   │
│  │  Assistant: "..."                         │   │
│  ├──────────────────────────────────────────┤   │
│  │ Current Response       (being generated)  │   │
│  └──────────────────────────────────────────┘   │
│                                                  │
│  Total capacity: 128K - 1M tokens (varies)       │
└─────────────────────────────────────────────────┘
```

---

## Context Window vs. Training Data

| | Context Window | Training Data |
|---|---|---|
| **What** | Working memory for current task | Everything the model learned |
| **Size** | 128K - 1M tokens | Trillions of tokens |
| **Content** | Your specific prompt, code, conversation | Internet, books, code |
| **Persistence** | Current session only | Permanent (baked in) |
| **Control** | You manage it | Model provider manages it |

---

## Why It Matters for Development

### The Good
- Larger windows can hold entire codebases for analysis
- Multi-file refactoring becomes possible in one session
- Conversation history provides continuity across turns

### The Bad
- Every token of irrelevant context dilutes attention
- As context grows, recall accuracy degrades (context rot)
- Token costs scale linearly with context size
- Exceeding the window causes errors or silent truncation

### The Reality
> "The context window is a precious resource. Treat every token like it costs money — because it does."

---

## Analogy: The Desk Metaphor

Think of the context window as a physical desk:
- **Small desk (32K tokens)** — Room for one document. You work on one thing at a time.
- **Medium desk (128K tokens)** — Room for a stack of papers. You can cross-reference.
- **Large desk (200K-1M tokens)** — Room for an entire filing cabinet. But finding anything requires effort.

**The key insight**: A bigger desk doesn't make you more organized. Without good organization, a bigger desk just means more clutter to search through.

---

## Key Takeaways

1. **Context window = working memory** — Everything the model can "see" right now
2. **It's finite** — Even 1M tokens has limits
3. **Quality over quantity** — The right 10K tokens outperform the wrong 100K
4. **It grows every turn** — Each message and tool result adds to the pile
5. **You control it** — What goes in the window is an engineering decision
