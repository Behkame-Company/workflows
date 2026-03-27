# Context Awareness

> Models that know how much context they have left — and why it changes everything.

---

## What Is Context Awareness?

Context awareness is a capability in Claude 4.5+ models that lets the model **track its remaining context window** throughout a conversation. It's like giving the model a clock for its working memory.

```
[Session start]
"You have 1,000,000 tokens available."

[After 50 tool calls]
"You have 723,451 tokens remaining."

[Getting low]
"You have 82,100 tokens remaining."
```

---

## Why It Matters

Without context awareness, the model is like a cook without a timer:
- Might try to do too much and run out mid-task
- Might cut work short because it doesn't know how much room is left
- Can't prioritize what to work on based on remaining capacity
- May start "wrapping up" prematurely or too late

With context awareness:
- **Sustained focus** — Model works persistently until near the limit
- **Graceful handoff** — Model prepares state notes before running out
- **Smart prioritization** — Tackles critical work first when context is low
- **No mid-sentence truncation** — Wraps up cleanly

---

## Leveraging Context Awareness in Prompts

### Tell the Model About Context Management
```markdown
## Context Management
You are working in an environment with context compaction.
When your context window runs low, your conversation history
will be summarized and you'll continue with fresh context.

Before compaction occurs:
1. Write progress notes to progress.txt
2. Commit any in-progress work to git
3. Document the current state of your task

After a fresh context window:
1. Read progress.txt and git log
2. Run basic verification tests
3. Continue from where you left off
```

### Pair with Memory Tool
```markdown
## Memory
You have a memory tool available. Use it to:
- Store architectural decisions
- Track task progress across sessions
- Save information you'll need after compaction
- Maintain a to-do list of remaining work
```

---

## Context Awareness + Multi-Window Workflows

```
Window 1 (1M tokens):
  ├── Setup environment, write tests
  ├── Implement feature A
  ├── [Context getting low]
  ├── Write progress notes
  └── Clean commit to git

Window 2 (fresh 1M tokens):
  ├── Read progress notes + git log
  ├── Verify feature A works
  ├── Implement feature B
  ├── [Context getting low]
  ├── Write progress notes
  └── Clean commit to git

Window 3 (fresh 1M tokens):
  └── ... continue pattern ...
```

---

## Which Models Support It

| Model | Context Awareness | Notes |
|-------|------------------|-------|
| Claude Opus 4.6 | ✅ | Full support |
| Claude Sonnet 4.6 | ✅ | Full support |
| Claude Sonnet 4.5 | ✅ | Full support |
| Claude Haiku 4.5 | ✅ | Full support |
| GPT-4o/4.1 | ❌ | Not available |
| Gemini | ❌ | Not available |

---

## Key Takeaways

1. **Context awareness = model knows its budget** — No more guessing
2. **Enables sustained work** — Model works to the end, not just the middle
3. **Pair with memory tool** — Persist state before compaction
4. **Prompt for behavior** — Tell the model what to do when context is low
5. **Multi-window workflows** — Each window picks up cleanly

---

## Next Steps

- 🔗 [What Is Compaction?](../04-compaction-and-summarization/what-is-compaction.md) — The complementary strategy
- 🔗 [Multi-Context Window Design](../06-multi-window-workflows/multi-context-window-design.md) — Long-running workflows
