# Context Window Management

> Master the finite resource that determines AI agent effectiveness — your context window.

---

## Why Context Window Management Matters

The context window is an AI model's "working memory." Every token of instructions, code, conversation history, and tool output competes for space in this finite resource. As Anthropic's research shows, **more context isn't automatically better** — as token count grows, accuracy and recall degrade through a phenomenon called *context rot*.

For AI-assisted development, this means:
- **Wasted context = worse code suggestions** — Irrelevant tokens dilute the model's attention
- **Overflowing context = lost state** — Long sessions lose track of earlier decisions
- **Poor context hygiene = compounding errors** — Stale information misleads the model
- **Context engineering > prompt engineering** — It's not just *what* you say, but *everything* in the window

---

## Who This Is For

- **Team leads** standardizing AI workflows
- **Developers** using Copilot CLI, Claude Code, or similar tools
- **AI engineers** building agentic systems and multi-turn workflows
- **Anyone** who's hit the wall of "the AI forgot what we were doing"

---

## Table of Contents

### 1. Fundamentals
- [What Is a Context Window?](01-fundamentals/what-is-a-context-window.md) — Core concepts and why it matters
- [Tokens and Tokenization](01-fundamentals/tokens-and-tokenization.md) — How text becomes tokens
- [Context Rot](01-fundamentals/context-rot.md) — Why more context can mean worse results
- [Context Window Comparison](01-fundamentals/context-window-comparison.md) — Model-by-model comparison

### 2. Context Engineering
- [From Prompt to Context Engineering](02-context-engineering/from-prompt-to-context-engineering.md) — The paradigm shift
- [Anatomy of Effective Context](02-context-engineering/anatomy-of-effective-context.md) — What makes good context
- [System Prompt Optimization](02-context-engineering/system-prompt-optimization.md) — Right altitude for instructions
- [Tool Design for Context Efficiency](02-context-engineering/tool-design-for-context-efficiency.md) — Token-efficient tools

### 3. Token Management
- [Token Counting & Budgeting](03-token-management/token-counting-and-budgeting.md) — Measuring and planning usage
- [Cost Optimization](03-token-management/cost-optimization.md) — Reducing token spend
- [Extended Thinking Tokens](03-token-management/extended-thinking-tokens.md) — How thinking affects context
- [Context Awareness](03-token-management/context-awareness.md) — Models that track their own budget

### 4. Compaction & Summarization
- [What Is Compaction?](04-compaction-and-summarization/what-is-compaction.md) — Core concept and why it matters
- [Server-Side Compaction](04-compaction-and-summarization/server-side-compaction.md) — Anthropic's compaction API
- [Manual Summarization Strategies](04-compaction-and-summarization/manual-summarization-strategies.md) — DIY context compression
- [Compaction Tuning](04-compaction-and-summarization/compaction-tuning.md) — Balancing recall vs. compression

### 5. Memory & Persistence
- [Memory Tool Guide](05-memory-and-persistence/memory-tool-guide.md) — Anthropic's memory tool
- [Structured Note-Taking](05-memory-and-persistence/structured-note-taking.md) — Agent self-documentation
- [Memory Bank Pattern](05-memory-and-persistence/memory-bank-pattern.md) — Cross-session knowledge bases
- [Cross-Session State](05-memory-and-persistence/cross-session-state.md) — Bridging context windows

### 6. Multi-Window Workflows
- [Multi-Context Window Design](06-multi-window-workflows/multi-context-window-design.md) — Architecture for long tasks
- [Initializer Agent Pattern](06-multi-window-workflows/initializer-agent-pattern.md) — Setting up the first session
- [Incremental Progress Pattern](06-multi-window-workflows/incremental-progress-pattern.md) — Feature-by-feature work
- [State Handoff Strategies](06-multi-window-workflows/state-handoff-strategies.md) — Clean transitions between windows

### 7. Context Retrieval
- [Just-in-Time Retrieval](07-context-retrieval/just-in-time-retrieval.md) — Loading data on demand
- [Hybrid Retrieval Strategies](07-context-retrieval/hybrid-retrieval-strategies.md) — Combining upfront + on-demand
- [Agentic Search](07-context-retrieval/agentic-search.md) — AI-driven context discovery
- [Progressive Disclosure](07-context-retrieval/progressive-disclosure.md) — Layer-by-layer context building

### 8. Sub-Agent Architectures
- [Context Isolation with Sub-Agents](08-sub-agent-architectures/context-isolation-with-subagents.md) — Clean context per task
- [Delegation Patterns](08-sub-agent-architectures/delegation-patterns.md) — When and how to delegate
- [Multi-Agent Context Coordination](08-sub-agent-architectures/multi-agent-context-coordination.md) — Orchestrating multiple agents

### 9. Best Practices
- [Minimal Viable Context](09-best-practices/minimal-viable-context.md) — The smallest set of high-signal tokens
- [Context Hygiene](09-best-practices/context-hygiene.md) — Keeping context clean over time
- [Debugging Context Issues](09-best-practices/debugging-context-issues.md) — When the AI goes off track
- [Tool-Specific Strategies](09-best-practices/tool-specific-strategies.md) — Copilot, Claude Code, Cursor

### 10. Anti-Patterns
- [Context Stuffing](10-anti-patterns/context-stuffing.md) — More is not better
- [The Stale Context Trap](10-anti-patterns/stale-context-trap.md) — Outdated information misleads
- [The Large Window Fallacy](10-anti-patterns/large-window-fallacy.md) — Big windows don't solve everything
- [Poor State Management](10-anti-patterns/poor-state-management.md) — When agents lose track

### 11. Practical Resources
- [Templates](11-practical/templates.md) — Ready-to-use configurations
- [Troubleshooting](11-practical/troubleshooting.md) — Common problems and fixes
- [FAQ](11-practical/faq.md) — Frequently asked questions
- [Resources](11-practical/resources.md) — Links and further reading

---

## Key Concepts at a Glance

| Concept | Definition |
|---------|-----------|
| **Context Window** | The total tokens a model can reference when generating a response |
| **Context Rot** | Degradation in accuracy as context length increases |
| **Context Engineering** | Curating the optimal set of tokens for inference |
| **Compaction** | Summarizing older context to free up space |
| **Memory Tool** | Persistent storage outside the context window |
| **Just-in-Time Retrieval** | Loading data on demand rather than upfront |
| **Progressive Disclosure** | Layer-by-layer context building through exploration |
| **Sub-Agent Isolation** | Dedicated context windows for focused tasks |

---

## Quick Start

1. **New to context windows?** Start with [What Is a Context Window?](01-fundamentals/what-is-a-context-window.md)
2. **Hitting context limits?** Read [Compaction](04-compaction-and-summarization/what-is-compaction.md) and [Memory](05-memory-and-persistence/memory-tool-guide.md)
3. **Building long-running agents?** See [Multi-Window Workflows](06-multi-window-workflows/multi-context-window-design.md)
4. **Optimizing existing workflows?** Check [Best Practices](09-best-practices/minimal-viable-context.md)
5. **Debugging AI confusion?** Read [Context Hygiene](09-best-practices/context-hygiene.md) and [Anti-Patterns](10-anti-patterns/context-stuffing.md)
