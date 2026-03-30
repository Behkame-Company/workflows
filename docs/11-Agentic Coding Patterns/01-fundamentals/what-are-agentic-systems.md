# What Are Agentic Systems?

> Understanding the spectrum from simple LLM calls to fully autonomous agents — and why the simplest solution is usually the best starting point.

---

## The Anthropic Definition

Anthropic's "[Building Effective Agents](https://www.anthropic.com/research/building-effective-agents)" research draws a critical distinction that cuts through industry hype:

- **Workflows**: Systems where LLMs are orchestrated through **predefined code paths** — the developer controls the flow.
- **Agents**: Systems where LLMs **dynamically direct their own processes** and tool usage — the model controls the flow.

Both fall under the umbrella of "agentic systems," but they differ fundamentally in who decides what happens next.

## The Complexity Spectrum

Agentic systems exist on a spectrum. Not everything needs to be a fully autonomous agent.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    AGENTIC SYSTEMS SPECTRUM                        │
│                                                                     │
│  Simple                                                   Complex  │
│  ◄─────────────────────────────────────────────────────────────►   │
│                                                                     │
│  ┌──────────┐  ┌──────────────┐  ┌───────────┐  ┌──────────────┐  │
│  │  Simple   │  │  Augmented   │  │           │  │              │  │
│  │ LLM Call  │──│    LLM       │──│ Workflow  │──│    Agent     │  │
│  │          │  │              │  │           │  │              │  │
│  └──────────┘  └──────────────┘  └───────────┘  └──────────────┘  │
│                                                                     │
│  "Summarize    "Answer using     "Chain three    "Explore repo,   │
│   this text"    codebase RAG"     LLM steps       decide what     │
│                                   with gates"      to change"     │
│                                                                     │
│  Who controls   Who controls      Who controls    Who controls     │
│  flow: N/A      flow: Code        flow: Code      flow: LLM       │
│                                                                     │
│  Predictability: ████████████████████████████████░░░░░░░░░░░░░░   │
│  Flexibility:    ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░████████████████   │
└─────────────────────────────────────────────────────────────────────┘
```

### Level 1: Simple LLM Call

A single prompt, a single response. No tools, no memory, no orchestration.

```typescript
// Level 1: Simple LLM call
const response = await llm.complete("Explain this error: " + errorMessage);
```

**Use when**: The task is self-contained and the model has all needed context in the prompt.

### Level 2: Augmented LLM

An LLM enhanced with retrieval, tools, and memory. The model can pull in context and take actions, but the overall flow is still a single turn.

```typescript
// Level 2: Augmented LLM with tools
const response = await llm.complete({
  prompt: "Fix the type error in user-service.ts",
  tools: [fileRead, fileWrite, shellExecute],
  context: await retrieveRelevantFiles("user-service.ts")
});
```

**Use when**: The task needs external context or tool use, but can be solved in one pass.

### Level 3: Workflow

Multiple LLM calls orchestrated through **predefined code paths**. The developer decides the sequence — which calls happen, in what order, with what gates.

```typescript
// Level 3: Workflow — developer controls the flow
const analysis = await llm.complete("Analyze this codebase for issues: " + code);
const isValid = await validateAnalysis(analysis);  // Programmatic gate

if (isValid) {
  const plan = await llm.complete("Create a fix plan: " + analysis);
  const implementation = await llm.complete("Implement fixes: " + plan);
}
```

**Use when**: The task decomposes into fixed, predictable steps with known control flow.

### Level 4: Agent

The LLM decides what to do next. It chooses which tools to call, in what order, and when to stop — operating in a loop until the task is complete.

```typescript
// Level 4: Agent — LLM controls the flow
while (!task.isComplete()) {
  const action = await llm.complete({
    prompt: "You are working on: " + task.description,
    tools: [fileRead, fileWrite, search, test, git],
    history: task.getHistory()
  });
  
  const result = await executeAction(action);
  task.addToHistory(result);
}
```

**Use when**: The task is open-ended, subtasks are unpredictable, and the model needs autonomy to explore and iterate.

## The Key Insight: Start Simple

From Anthropic's research:

> *"Find the simplest solution possible, and only increase complexity when needed."*

This is the most important principle in agentic system design. The industry tendency is to reach for complex agent frameworks immediately, but:

- **Most successful implementations use simple, composable patterns** — not sprawling frameworks
- For many applications, **optimizing a single LLM call** with retrieval and in-context examples is sufficient
- Adding agentic complexity introduces **latency, cost, and compounding error risk**

```
┌────────────────────────────────────────────────────┐
│         DECISION: WHAT LEVEL DO I NEED?            │
│                                                    │
│  Can a single prompt solve it?                     │
│  ├── YES → Use a simple LLM call                  │
│  └── NO                                            │
│      Does it need external context or tools?       │
│      ├── YES (single turn) → Augmented LLM        │
│      └── YES (multi-step)                          │
│          Are the steps predictable?                │
│          ├── YES → Workflow                        │
│          └── NO → Agent                            │
└────────────────────────────────────────────────────┘
```

## Workflows vs. Agents: The Tradeoff

| Dimension        | Workflows                          | Agents                               |
| ---------------- | ---------------------------------- | ------------------------------------ |
| **Control flow** | Predefined by developer            | Dynamic, decided by LLM             |
| **Predictability** | High — same input, same path     | Lower — path varies per run          |
| **Debuggability** | Easy — inspect each step          | Harder — emergent behavior           |
| **Cost**          | Predictable — fixed number of calls | Variable — depends on task complexity |
| **Error handling** | Explicit gates between steps      | Model must self-correct              |
| **Best for**      | Well-defined, repeatable processes | Open-ended, exploratory tasks        |

## Real-World Examples in Coding

| Task                        | Appropriate Level | Why                                          |
| --------------------------- | ----------------- | -------------------------------------------- |
| Generate a commit message   | Simple LLM call   | All context fits in one prompt               |
| Explain a code block        | Augmented LLM     | Needs file context, but single-turn          |
| Migrate API from v2 to v3   | Workflow           | Predictable steps: find → plan → change → test |
| "Fix all bugs in this repo" | Agent              | Unpredictable scope, needs exploration       |
