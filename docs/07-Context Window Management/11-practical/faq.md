# FAQ

> Frequently asked questions about context window management.

---

## Basics

### Q: What's a context window?
All the tokens the model can see when generating a response — system prompt, conversation history, tool results, and the response itself.

### Q: How big is a context window?
Depends on the model: 128K tokens (GPT-4o), 200K tokens (Claude Sonnet 4), 1M tokens (Claude Opus 4.6, GPT-4.1). See [Context Window Comparison](../01-fundamentals/context-window-comparison.md).

### Q: How many lines of code fit in a context window?
Roughly: 1K tokens ≈ 100-150 lines of code. So 200K tokens ≈ 20K-30K lines. But quality degrades well before the limit.

### Q: Does the model's response count toward the context window?
**Yes.** Input + output together must fit in the window. Always reserve space for the response.

---

## Context Rot

### Q: Does quality really degrade with more context?
**Yes.** This is empirically demonstrated. At 200K tokens, recall accuracy is notably lower than at 10K tokens. See [Context Rot](../01-fundamentals/context-rot.md).

### Q: Is the "lost in the middle" effect real?
**Yes.** Models recall information better from the start and end of context than from the middle. Put critical instructions at the start, recent context at the end.

### Q: Can I just use a bigger window to avoid context rot?
**No.** Context rot scales with context size. A 1M window at 100% utilization has worse recall than a 200K window at 50%. See [The Large Window Fallacy](../10-anti-patterns/large-window-fallacy.md).

---

## Compaction

### Q: What is compaction?
Summarizing older parts of a conversation to free up space for new work. Can be automatic (server-side) or manual. See [What Is Compaction?](../04-compaction-and-summarization/what-is-compaction.md).

### Q: When should compaction trigger?
Set the trigger at **75% of your window size**. For a 200K window, trigger at 150K.

### Q: Does compaction lose information?
It summarizes, so some detail is lost. Critical information should also be stored in persistent files (progress.txt, decisions.md) as insurance.

### Q: Can I customize what compaction preserves?
**Yes.** Anthropic's server-side compaction accepts custom summarization instructions. See [Compaction Tuning](../04-compaction-and-summarization/compaction-tuning.md).

---

## Memory

### Q: How do I persist information across sessions?
Use the memory tool, memory bank files, CLAUDE.md, or progress.txt files written to disk. See [Memory & Persistence](../05-memory-and-persistence/).

### Q: What's the Memory Bank pattern?
A structured set of markdown files (projectbrief, techContext, activeContext, progress) that the AI reads at session start. Popularized by Cline. See [Memory Bank Pattern](../05-memory-and-persistence/memory-bank-pattern.md).

### Q: Should I use the memory tool or files on disk?
Both work. Memory tool is more structured; files on disk are simpler and version-controlled with git. For most development projects, files on disk are sufficient.

---

## Multi-Window

### Q: What if my task exceeds one context window?
Use the multi-window pattern: initializer agent sets up scaffolding, coding agent works incrementally. See [Multi-Context Window Design](../06-multi-window-workflows/multi-context-window-design.md).

### Q: Fresh window or compaction between sessions?
For development tasks, **fresh windows with state artifacts** often outperform compaction, because models excel at discovering state from files and git.

---

## Cost

### Q: How do I reduce token costs?
Cache static content, use targeted file reads, compress tool outputs, use cheaper models for simple tasks, and avoid context stuffing. See [Cost Optimization](../03-token-management/cost-optimization.md).

### Q: Are thinking tokens expensive?
**Yes.** Thinking tokens are billed at output rates (3-5x input cost). Use lower effort for straightforward tasks. See [Extended Thinking Tokens](../03-token-management/extended-thinking-tokens.md).
