# Context Rot

> Why more context can mean worse results — the degradation curve every developer needs to understand.

---

## What Is Context Rot?

**Context rot** is the empirical observation that as the number of tokens in the context window increases, the model's ability to accurately recall and reason about information in that context decreases.

This happens because of the **n² attention problem**: every token attends to every other token. As n grows, the model's ability to capture these pairwise relationships gets stretched thin.

```
Token Count     Recall Accuracy (approximate)
──────────────  ───────────────────────────────
  1K tokens     ████████████████████  ~99%
 10K tokens     ██████████████████░░  ~95%
 50K tokens     ████████████████░░░░  ~90%
100K tokens     ██████████████░░░░░░  ~85%
200K tokens     ████████████░░░░░░░░  ~78%
500K tokens     ██████████░░░░░░░░░░  ~70%
  1M tokens     ████████░░░░░░░░░░░░  ~65%
```

*These are illustrative — actual curves vary by model, task, and content placement.*

---

## Why Context Rot Matters for Coding

### Scenario: Refactoring with Full Codebase in Context
```
Small project (10K tokens):  AI remembers all file relationships  ✅
Medium project (50K tokens): AI sometimes misses cross-file deps  ⚠️
Large project (200K tokens): AI loses track of earlier files       ❌
Full monorepo (500K+ tokens): AI hallucinates connections          💀
```

### Real-World Symptoms
- AI "forgets" earlier instructions as conversation grows
- AI generates code that contradicts patterns from earlier in the context
- AI fails to reference relevant files that are deep in the history
- AI starts repeating itself or losing track of what was done

---

## The Position Effect

Where information appears in the context matters:

```
┌──────────────────────────────────────────┐
│  START (strong recall)                   │  ← System prompt, instructions
│                                          │
│  MIDDLE (weakest recall — "lost middle") │  ← Earlier conversation turns
│                                          │
│  END (strong recall)                     │  ← Recent messages, current query
└──────────────────────────────────────────┘
```

This "U-shaped" recall curve means:
- Instructions at the start are well-remembered
- The most recent context is well-remembered
- **Information in the middle gets lost**

---

## Mitigation Strategies

| Strategy | How It Helps |
|----------|-------------|
| **Compaction** | Summarizes old context, freeing attention budget |
| **Just-in-time retrieval** | Only load what's needed right now |
| **Sub-agents** | Isolated context windows per task |
| **Structured notes** | Key info persisted outside context |
| **Recency bias** | Put critical info near the end |
| **Periodic reminders** | Re-state key instructions mid-conversation |

---

## Key Takeaways

1. **Context rot is real** — More tokens = less accurate recall
2. **The middle gets lost** — Critical info should be at start or end
3. **Don't dump everything in** — Curate what the model sees
4. **Smaller, focused context wins** — 10K relevant tokens > 100K mixed tokens
5. **Engineer around it** — Compaction, retrieval, sub-agents are your tools

---

## Next Steps

- 🔗 [Context Window Comparison](context-window-comparison.md) — Model capacities
- 🔗 [Minimal Viable Context](../09-best-practices/minimal-viable-context.md) — The solution
