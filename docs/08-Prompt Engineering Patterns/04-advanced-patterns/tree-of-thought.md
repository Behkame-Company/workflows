# Tree of Thought

> Branching exploration with evaluation and backtracking — for problems that need search.

---

## What Is Tree of Thought?

Tree of Thought (ToT) extends Chain-of-Thought by exploring multiple reasoning branches at each step, evaluating which branches are promising, and backtracking from dead ends.

```
Problem
├── Approach A
│   ├── Step A1 ✓ (promising)
│   │   ├── Step A1a ✓ → Solution!
│   │   └── Step A1b ✗ (dead end)
│   └── Step A2 ✗ (dead end, backtrack)
├── Approach B
│   └── Step B1 ~ (maybe, explore further)
└── Approach C ✗ (eliminated early)
```

---

## How It Differs from CoT

| Feature | Chain-of-Thought | Tree of Thought |
|---------|-----------------|-----------------|
| Paths | Single linear | Multiple branching |
| Evaluation | At the end | At each step |
| Backtracking | No | Yes |
| Cost | Low | High (5-10x) |
| Best for | Sequential reasoning | Search/exploration problems |

---

## When to Use ToT

✅ **Good use cases**:
- Choosing between multiple architectural approaches
- Complex refactoring where multiple strategies exist
- Finding the optimal database schema design
- Solving performance bottlenecks with multiple causes

❌ **Overkill for**:
- Simple code generation
- Formatting tasks
- Clear, single-path tasks

---

## Implementation: Simplified Prompt-Based ToT

You can approximate ToT in a single prompt:

```
I need to choose the best approach for [problem].

Consider 3 different approaches:

Approach 1: [First idea]
- Evaluate: What are the pros? What are the risks?
- Score: promising / maybe / unlikely

Approach 2: [Second idea]
- Evaluate: What are the pros? What are the risks?
- Score: promising / maybe / unlikely

Approach 3: [Third idea]
- Evaluate: What are the pros? What are the risks?
- Score: promising / maybe / unlikely

Now take the most promising approach and develop it further:
- What are the implementation steps?
- What could go wrong?
- How do we mitigate those risks?

Final recommendation with reasoning.
```

---

## ToT for Architecture Decisions

```
We need to implement real-time notifications. Explore these branches:

Branch 1: WebSockets (Socket.IO)
  → Evaluate: complexity, scalability, client compatibility
  → If promising, detail: implementation plan, server costs, failure modes

Branch 2: Server-Sent Events
  → Evaluate: simplicity, limitations, browser support
  → If promising, detail: implementation plan, polling fallback

Branch 3: Polling with HTTP
  → Evaluate: simplicity, latency, server load
  → If promising, detail: interval optimization, caching strategy

For the winning approach, provide:
1. Implementation outline
2. Key risks and mitigations
3. Estimated complexity (person-days)
```

---

## Cost Management

ToT is expensive. Use it strategically:

| Strategy | Tokens | Reliability |
|----------|--------|-------------|
| Full ToT (3 branches, 3 deep) | 10-20x | Highest |
| Simplified ToT (prompt-based) | 2-3x | Good |
| Breadth-first (evaluate all, expand best) | 3-5x | Good |
| Human-guided (you pick which branch to expand) | 2x per step | Flexible |

---

## Next Steps

- 🔗 [Reflection & Critique](reflection-and-critique.md) — Evaluate and improve solutions
- 🔗 [Decomposition](decomposition.md) — Breaking problems into parts
