# Start Simple

> Anthropic's #1 principle for building with agents: start with the simplest approach, measure outcomes, and add complexity only when it demonstrably improves results.

---

## The Complexity Ladder

Most successful agent implementations follow the same path: they start simple and add complexity only when measured outcomes justify it.

```
┌──────────────────────────────────────────────────────────────┐
│              The Complexity Ladder                            │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│   Level 4: AUTONOMOUS AGENT                                  │
│   ████████████████████████████████████                       │
│   Self-directed, multi-step, tool-using                      │
│   Use when: task requires adaptive decision-making           │
│                                                              │
│   Level 3: WORKFLOW                                          │
│   ██████████████████████████████                             │
│   Orchestrated chain of LLM calls                            │
│   Use when: task has defined steps needing LLM at each       │
│                                                              │
│   Level 2: AUGMENTED LLM                                     │
│   ████████████████████████                                   │
│   Single LLM call + retrieval or tools                       │
│   Use when: LLM needs external data or actions               │
│                                                              │
│   Level 1: SINGLE LLM CALL                                   │
│   ██████████████████                                         │
│   One prompt, one response                                   │
│   Use when: task is self-contained with clear instructions   │
│                                                              │
│   ▲ Add complexity ONLY when lower level is insufficient     │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

## The Decision Process

Before reaching for agents, work through this checklist:

### Step 1: Try a Single Call

```
Task: "Generate a Zod validation schema for the User model"

Approach: One prompt with clear instructions
───────────────────────────────────────────
Prompt: "Given this TypeScript interface, generate a Zod
validation schema with appropriate constraints:
interface User { id: string; email: string; age: number; }"

Result: Usually sufficient. Move on.
```

### Step 2: If Not Enough, Add Retrieval/Tools

```
Task: "Generate a Zod schema that matches our existing patterns"

Why single call isn't enough: Agent doesn't know your patterns
───────────────────────────────────────────────────────────────
Add: RAG retrieval of existing schemas in the codebase
     OR give the agent file-read tools to explore patterns

Result: Agent reads existing schemas, follows your patterns.
```

### Step 3: If Still Not Enough, Try a Workflow

```
Task: "Generate schemas for all 12 models and ensure consistency"

Why augmented LLM isn't enough: Too many steps for one call
───────────────────────────────────────────────────────────
Workflow: For each model → read interface → generate schema
          → validate against existing schemas → write file

Result: Orchestrated chain handles the multi-step process.
```

### Step 4: Only Then Consider Agents

```
Task: "Refactor the entire validation layer, handling unknown
edge cases and adapting to what you find"

Why workflow isn't enough: Can't predict the steps in advance
─────────────────────────────────────────────────────────────
Agent: Explores codebase, decides approach, implements,
       tests, iterates on failures, adapts to findings

Result: Agent handles the adaptive, open-ended nature.
```

## Measuring "Not Enough"

Don't guess — measure:

```
┌──────────────────────────────────────────────────────────────┐
│              Before Adding Complexity                         │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│   1. Define success criteria FIRST                           │
│      "Schema validates all edge cases in our test suite"     │
│                                                              │
│   2. Measure current approach                                │
│      "Single prompt gets 8/10 test cases right"              │
│                                                              │
│   3. Add one level of complexity                             │
│      "With file reading, gets 10/10 test cases"              │
│                                                              │
│   4. Measure again                                           │
│      Did outcomes improve enough to justify the cost?        │
│                                                              │
│   Cost of complexity:                                        │
│   • More tokens (= more money)                               │
│   • More latency (= slower feedback)                         │
│   • More failure modes (= harder to debug)                   │
│   • More maintenance (= ongoing effort)                      │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

## The Framework Trap

Anthropic's explicit warning:

> "Frameworks often create extra layers of abstraction that can obscure the underlying prompts and responses, making them harder to debug."

```
┌──────────────────────────────────────────────────────────────┐
│           Framework Abstraction Layers                        │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│   What you want:                                             │
│   Prompt ──▶ LLM ──▶ Response                                │
│                                                              │
│   What frameworks add:                                       │
│   Prompt ──▶ Framework ──▶ Wrapper ──▶ Adapter ──▶ LLM      │
│          ◀── Framework ◀── Parser ◀── Adapter ◀──            │
│                                                              │
│   Each layer:                                                │
│   • Adds latency                                             │
│   • Adds potential failure points                            │
│   • Obscures what's actually being sent to the LLM          │
│   • Makes debugging harder                                   │
│   • Creates vendor lock-in                                   │
│                                                              │
│   Direct API calls are simpler, faster, and clearer          │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

This doesn't mean never use frameworks. It means:
1. **Understand what the framework does** before using it
2. **Start without the framework** — use direct API calls
3. **Add the framework** only when you've proven you need its features
4. **Be able to debug** through the framework's abstractions

## Common Mistake: Premature Agent Architecture

```
Scenario: Team needs to automate code review

❌ Over-engineered approach (Day 1):
   • Multi-agent system with orchestrator
   • Security agent, style agent, test agent
   • Custom framework for agent communication
   • Result database for review aggregation
   → 3 weeks to build, fragile, hard to debug

✓ Simple approach (Day 1):
   • Single LLM call in CI with the diff as context
   • Good prompt with review checklist
   → 2 hours to build, immediately useful

   Then iterate:
   • Week 2: Add file-reading for more context
   • Month 2: Add separate security-focused prompt
   • Month 6: Consider multi-agent only if needed
```

## When Simple Really Is Enough

Many tasks that seem to need agents actually don't:

| Task | Agent Needed? | Simple Alternative |
|---|---|---|
| Generate boilerplate code | No | Single prompt with template |
| Explain a function | No | Single prompt with code context |
| Review a small PR | No | Single prompt with diff |
| Translate code between languages | No | Single prompt with source |
| Write test cases | Maybe | Prompt + test framework docs |
| Implement a feature | Maybe | Depends on scope and complexity |
| Debug a production issue | Yes | Requires exploration and iteration |
| Refactor a module | Yes | Requires analysis and adaptation |

## The Simplicity Checklist

Before building any agent system, verify:

- [ ] **I've defined success criteria** — I know what "good" looks like
- [ ] **I've tried a single prompt** — and measured the results
- [ ] **I've tried adding context** — retrieval or file reading
- [ ] **I've tried a simple workflow** — orchestrated steps
- [ ] **Adding an agent measurably improves outcomes** — not just feels right
- [ ] **I understand the failure modes** — and have recovery strategies
- [ ] **The benefit justifies the cost** — tokens, latency, maintenance
