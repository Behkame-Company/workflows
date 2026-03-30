# Context Isolation with Sub-Agents

> Dedicated context windows for focused tasks — cleaner context, better results.

---

## The Problem

A single agent handling a complex task accumulates context from every sub-task:

```
Single agent context after investigating 5 files:
  File A contents: 1,500 tokens  (no longer needed)
  File B contents: 1,200 tokens  (no longer needed)
  File C contents: 2,000 tokens  (no longer needed)
  File D contents: 800 tokens    (still relevant)
  File E contents: 1,500 tokens  (still relevant)
  Total waste: 4,700 tokens of stale context
```

---

## The Solution: Sub-Agents

Instead of one agent doing everything, delegate focused tasks to sub-agents with clean context windows:

```
┌───────────────────────────────────────────┐
│  LEAD AGENT (orchestrator)                │
│  Context: High-level plan + summaries     │
│                                           │
│  "I need to understand the auth module,   │
│   fix the bug, and update tests."         │
└────────┬──────────┬──────────┬────────────┘
         │          │          │
    ┌────▼────┐┌────▼────┐┌───▼─────┐
    │Sub-Agent││Sub-Agent││Sub-Agent│
    │"Explore ││"Fix bug ││"Update  │
    │ auth    ││ in jwt" ││ tests"  │
    │ module" ││         ││         │
    │         ││         ││         │
    │Returns: ││Returns: ││Returns: │
    │200-token││Patch    ││Test     │
    │summary  ││file     ││file     │
    └─────────┘└─────────┘└─────────┘
```

Each sub-agent:
- Starts with a **clean context** — no stale data
- Handles a **focused task** — deep investigation
- Returns a **compressed result** — summary, not raw data
- Uses up to **100K tokens internally** — returns 1-2K tokens

---

## Context Isolation Benefits

| Without Sub-Agents | With Sub-Agents |
|--------------------|--------------------|
| Lead agent sees all raw data | Lead agent sees only summaries |
| Context grows linearly | Context stays flat |
| Stale results accumulate | Each task has fresh context |
| Attention diluted across everything | Focused attention per task |

---

## When to Use Sub-Agents

| Scenario | Sub-Agent? | Why |
|----------|-----------|-----|
| Quick file read | ❌ No | Overhead isn't worth it |
| Deep investigation across multiple files | ✅ Yes | Isolates exploration context |
| Research task (web search, docs) | ✅ Yes | Research generates lots of context |
| Code generation | ✅ Yes | Focused on one task |
| Simple refactoring | ❌ No | Direct is faster |
| Complex multi-file refactoring | ✅ Yes | Each file gets focused attention |

**Rule of thumb**: If the task will generate >5K tokens of intermediate context that the lead agent doesn't need, use a sub-agent.

---

## Orchestration Pattern

```python
# Lead agent prompt
"""
You are a lead agent coordinating a complex task.
You have sub-agents available for focused work.

When a task requires deep investigation or generates
lots of intermediate context, delegate to a sub-agent.

Sub-agents return concise summaries. Use these summaries
to make decisions and coordinate the overall task.
"""

# Sub-agent prompt
"""
You are a focused agent investigating [specific task].
Explore thoroughly, then return a concise summary of:
1. What you found
2. Key decisions or recommendations
3. Relevant file paths and line numbers

Keep your summary under 2,000 tokens.
"""
```

---

## Key Takeaways

1. **Sub-agents have clean context** — No stale data from other tasks
2. **Lead agent stays lightweight** — Only summaries, not raw data
3. **100K internal → 2K returned** — Massive compression ratio
4. **Not for simple tasks** — Overhead isn't worth it for quick reads
5. **Natural separation of concerns** — Explore, implement, test separately
