# What Is Compaction?

> The core strategy for managing conversations that exceed context window limits.

---

## Definition

**Compaction** is the practice of taking a conversation nearing the context window limit, summarizing its contents, and continuing with the compressed version. It's like writing meeting notes instead of keeping a verbatim transcript.

```
Before compaction (150K tokens):
  Turn 1: [Full message — 2K tokens]
  Turn 2: [Full message — 5K tokens]  ← Tool results
  Turn 3: [Full message — 3K tokens]
  ...
  Turn 50: [Full message — 4K tokens]

After compaction (30K tokens):
  [Summary of turns 1-45 — 5K tokens]
  Turn 46: [Full message — 4K tokens]  ← Recent turns preserved
  Turn 47: [Full message — 3K tokens]
  Turn 48: [Full message — 5K tokens]
  Turn 49: [Full message — 4K tokens]
  Turn 50: [Full message — 4K tokens]
```

---

## Types of Compaction

### 1. Server-Side Compaction (Anthropic)
- Automatic, built into the API
- Model summarizes its own conversation
- Transparent to the user
- Currently supported on Claude 4.6 models

### 2. Manual Compaction
- Developer-implemented summarization
- More control over what's preserved
- Works with any model
- Requires engineering effort

### 3. Tool Result Clearing
- Lightest form of compaction
- Replace old tool outputs with summaries
- Preserves conversation structure
- Simple to implement

---

## When to Compact

| Signal | Action |
|--------|--------|
| 75% of context used | Prepare for compaction |
| 85% of context used | Compact now |
| Model starts forgetting instructions | Compact + re-inject key instructions |
| Tool results dominate context | Clear old tool results |
| Session will continue for many more turns | Proactively compact |

---

## The Compaction Trade-Off

Aggressive compaction frees room but risks losing critical context; conservative compaction preserves detail but leaves less headroom. See [Compaction Tuning](compaction-tuning.md) for the Recall-Compression Spectrum and tuning strategies.

---

## Key Takeaways

1. **Compaction = summarize and continue** — Don't lose the thread
2. **Server-side is easiest** — Anthropic handles it automatically
3. **Tool result clearing is cheapest** — Start there
4. **Balance recall vs. compression** — Don't over-compress
5. **Compact before you run out** — Proactive > reactive
