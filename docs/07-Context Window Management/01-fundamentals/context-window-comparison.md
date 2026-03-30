# Context Window Comparison

> Model-by-model comparison of context window sizes, features, and practical limits.

---

## Model Comparison (2025-2026)

> **Note**: Model capabilities and context window sizes change frequently. Verify current specifications at each provider's official documentation.

| Model | Context Window | Effective Limit | Context Awareness | Compaction |
|-------|---------------|----------------|-------------------|------------|
| **Claude Opus 4.6** | 1M tokens | ~800K practical | ✅ Yes | ✅ Server-side |
| **Claude Sonnet 4.6** | 1M tokens | ~800K practical | ✅ Yes | ✅ Server-side |
| **Claude Sonnet 4.5** | 200K (1M beta) | ~150K practical | ✅ Yes | ❌ |
| **Claude Haiku 4.5** | 200K tokens | ~150K practical | ✅ Yes | ❌ |
| **GPT-4o** | 128K tokens | ~100K practical | ❌ No | ❌ |
| **GPT-4.1** | 1M tokens | ~800K practical | ❌ No | ❌ |
| **Gemini 2.5 Pro** | 1M tokens | ~800K practical | ❌ No | ❌ |

**Effective limit** = Practical maximum before context rot noticeably degrades quality.

---

## What "Context Window Size" Actually Means

```
Total Context Window = Input Tokens + Output Tokens

Where Input Tokens include:
  - System prompt
  - Tool definitions
  - Conversation history (all turns)
  - Tool results
  - Current user message

And Output Tokens include:
  - Thinking tokens (if extended thinking enabled)
  - Response text
  - Tool calls
```

**Important**: The output is part of the window. A 200K window with a 10K response leaves 190K for input.

---

## Practical Capacity Guide

| Use Case | Tokens Needed | Minimum Model |
|----------|--------------|---------------|
| Quick code question | 2K-5K | Any model |
| File review + suggestions | 5K-15K | Any model |
| Multi-file refactoring | 20K-50K | 128K+ model |
| Full project analysis | 50K-200K | 200K+ model |
| Codebase migration | 200K-500K | 1M model |
| Long agentic session (hours) | 500K+ | 1M + compaction |

---

## Context Awareness Feature

Claude 4.5+ models have **context awareness** — the model knows how many tokens remain:

```
[At start]: "You have 1,000,000 tokens available."
[After tool calls]: "You have 847,231 tokens remaining."
```

This enables:
- Model self-manages context budget
- Wraps up work gracefully before running out
- Prioritizes critical work when context is low
- Avoids mid-sentence truncation

---

## Key Takeaways

1. **Window size ≠ usable capacity** — Reserve space for output and tool definitions
2. **Context awareness matters** — Models that track their budget work better long-term
3. **Compaction extends effective capacity** — Enables sessions beyond raw window size
4. **Practical limit < advertised limit** — Context rot starts well before the boundary
5. **Match model to task** — Don't use a 128K model for a 200K task
