# When to Go Agentic

> A practical decision framework for choosing the right level of complexity — because agentic systems trade latency and cost for better task performance, and that tradeoff isn't always worth it.

---

## The Core Tradeoff

For the complete agentic systems spectrum, see [What Are Agentic Systems](what-are-agentic-systems.md). This guide focuses on the decision framework.

Every step up the complexity spectrum trades **latency, cost, and error surface** for better task performance. The question isn't "should I build an agent?" It's **"does the performance gain justify the added complexity?"**

## The Decision Tree

Use this framework to find the right level for your task:

```
┌─────────────────────────────────────────────────────────────┐
│              WHEN TO GO AGENTIC: DECISION TREE              │
│                                                             │
│  Start here: Can a single LLM call solve this?             │
│  │                                                          │
│  ├── YES ──────────────────────► SIMPLE LLM CALL            │
│  │   "Generate a commit message"   Stop here. Don't         │
│  │   "Explain this function"       over-engineer it.        │
│  │                                                          │
│  └── NO: Needs multiple steps or external context           │
│      │                                                      │
│      ├── Single turn with tools? ──► AUGMENTED LLM          │
│      │   "Answer using codebase"     One call + retrieval   │
│      │   "Find and read the config"  + tools is enough.     │
│      │                                                      │
│      └── Needs multi-step processing                        │
│          │                                                  │
│          ├── Steps are fixed/known? ──► WORKFLOW             │
│          │   "Migrate tests from                            │
│          │    Jest to Vitest"          Use prompt chaining,  │
│          │   "Lint → Fix → Test"      routing, or parallel. │
│          │                                                  │
│          └── Steps are dynamic/unpredictable                │
│              │                                              │
│              ├── Has clear eval criteria? ► EVAL-OPTIMIZER   │
│              │   "Make this code pass                       │
│              │    all review checks"     Loop until quality  │
│              │                           criteria are met.  │
│              │                                              │
│              └── Truly open-ended? ────► AUTONOMOUS AGENT   │
│                  "Fix all the bugs"      LLM decides what   │
│                  "Build this feature"    to do and when      │
│                                          to stop.           │
└─────────────────────────────────────────────────────────────┘
```

## Five Questions to Ask

Before reaching for agentic patterns, systematically work through these questions:

### 1. Can a Single LLM Call Solve This?

**Try this first.** You'd be surprised how much a well-crafted prompt with good context can accomplish in one shot.

```typescript
// Before: Assumed you need a multi-step agent
const agent = new CodingAgent({ tools, maxSteps: 20 });
await agent.run("Add input validation to the signup form");

// After: One well-crafted call might be enough
const response = await llm.complete({
  system: "You are an expert TypeScript developer.",
  prompt: `Add input validation to the signup form in src/components/SignupForm.tsx.
           
           Current file contents:
           ${await readFile("src/components/SignupForm.tsx")}
           
           Validation rules:
           - Email: valid format, required
           - Password: min 8 chars, one uppercase, one number
           - Name: required, 2-50 chars
           
           Return the complete updated file.`
});
```

**Optimize the single call first**: better prompts, retrieval, in-context examples. Only escalate if this genuinely isn't enough.

### 2. Is the Task Decomposable into Fixed Steps?

If you can write down the steps in advance, use a **workflow** (prompt chaining, routing, or parallelization).

```typescript
// Fixed steps → Workflow (prompt chaining)
async function migrateTestFile(filePath: string) {
  // Step 1: Analyze current test structure
  const analysis = await llm.complete(
    `Analyze this Jest test file and identify all Jest-specific APIs used:\n${await readFile(filePath)}`
  );
  
  // Gate: Validate analysis
  if (!analysis.includes("Jest APIs found")) {
    return { status: "skipped", reason: "No Jest APIs detected" };
  }
  
  // Step 2: Generate migration plan
  const plan = await llm.complete(
    `Create a migration plan from Jest to Vitest for:\n${analysis}`
  );
  
  // Step 3: Execute migration
  const migrated = await llm.complete(
    `Apply this migration plan and return the complete updated file:\n${plan}\n\nOriginal:\n${await readFile(filePath)}`
  );
  
  return { status: "migrated", content: migrated };
}
```

### 3. Are the Subtasks Unpredictable?

If you can't know in advance what steps are needed — use **orchestrator-workers**.

```typescript
// Unpredictable subtasks → Orchestrator-Workers
// "Refactor the authentication system" — which files? how many changes?
const orchestrator = await llm.complete({
  prompt: `Analyze this task and determine what needs to change:
           Task: Refactor authentication from session-based to JWT
           Codebase structure: ${await getProjectTree()}`,
  tools: [searchCode, readFile]
});
// Orchestrator decides: "Need to change 7 files, create 2 new ones"
// Then delegates each to a worker
```

### 4. Does the Task Need Iterative Refinement?

If there are clear quality criteria and the output improves with feedback — use **evaluator-optimizer**.

```typescript
// Clear eval criteria + iterative improvement → Evaluator-Optimizer
async function generateWithQuality(spec: string) {
  let code = await generate(spec);
  
  for (let i = 0; i < 3; i++) {
    const review = await evaluate(code, spec);
    if (review.passes) return code;
    code = await refine(code, review.feedback);
  }
  
  return code;
}
```

### 5. Is This Truly Open-Ended?

Only use a full autonomous agent when the task genuinely requires exploration, judgment, and dynamic decision-making.

```typescript
// Truly open-ended → Autonomous Agent
// "Investigate why the app is slow and fix the main bottlenecks"
// The agent needs to: profile, analyze, hypothesize, experiment, verify
```

## Cost-Benefit Analysis

| Level | Latency | Cost | Error Risk | When Worth It |
|---|---|---|---|---|
| **Simple call** | ~2s | 1 call | Lowest | Always try first |
| **Augmented LLM** | ~5s | 1 call + retrieval | Low | Task needs external context |
| **Workflow** | ~10-30s | 3-5 calls | Medium | Fixed steps improve quality |
| **Orchestrator** | ~30-60s | 5-20 calls | Higher | Dynamic subtask decomposition needed |
| **Agent** | ~1-5min | 10-50+ calls | Highest | Open-ended exploration required |

### The Compounding Error Problem

Each LLM call has some probability of error. In a chain, errors compound:

```
Single call:    95% accuracy → 95% success
3-step chain:   95% × 95% × 95% = 85.7% success  
10-step agent:  95%^10 = 59.9% success
20-step agent:  95%^20 = 35.8% success
```

This is why **gates** (validation steps between calls) and **evaluation loops** are critical — they catch and correct errors before they propagate.

## Coding Examples: Matching Tasks to Levels

### Simple Code Changes → Single LLM Call

```typescript
// Rename a variable, fix a typo, add a comment, generate a type
const result = await llm.complete(
  `Add JSDoc comments to all exported functions in:\n${fileContent}`
);
```

### Multi-File Refactor → Workflow

```typescript
// Step 1: Find all files using the old pattern
// Step 2: Generate changes for each file  
// Step 3: Apply changes
// Step 4: Run tests to verify
// Each step is predictable — workflow is perfect
```

### Complex Feature → Orchestrator-Workers

```typescript
// "Add user authentication with OAuth"
// Orchestrator analyzes: need routes, middleware, database schema, 
// frontend components, tests — delegates each to focused workers
```

### Bug Investigation → Agent

```typescript
// "The app crashes when uploading large files"
// Agent needs to: read logs, reproduce, trace code paths,
// form hypotheses, test fixes — inherently exploratory
```

## The "Start Simple" Principle in Practice

```
┌─────────────────────────────────────────────────────┐
│            THE ITERATIVE APPROACH                    │
│                                                      │
│  1. Try a single, well-crafted LLM call             │
│     └── Good enough? → Ship it                      │
│                                                      │
│  2. Add retrieval and tools (augmented LLM)          │
│     └── Good enough? → Ship it                      │
│                                                      │
│  3. Add a second step with a gate (prompt chain)     │
│     └── Good enough? → Ship it                      │
│                                                      │
│  4. Add routing or parallelization                   │
│     └── Good enough? → Ship it                      │
│                                                      │
│  5. Add dynamic orchestration                        │
│     └── Good enough? → Ship it                      │
│                                                      │
│  6. Build a full agent loop                          │
│     └── This should be rare                         │
└─────────────────────────────────────────────────────┘
```

Don't start at step 6. Start at step 1 and only escalate when you have evidence that the simpler approach falls short.
