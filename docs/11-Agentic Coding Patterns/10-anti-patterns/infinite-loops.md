# Anti-Pattern: Infinite Agent Loops

> Agents stuck in retry cycles that burn tokens without making progress — and how to detect, prevent, and break out of them.

---

## Why Agents Loop

Agentic systems are iterative by design — they try, observe, adjust, and retry. But without safeguards, this loop becomes infinite. The agent doesn't know it's stuck; it just keeps trying.

```
Normal Agent Loop:              Infinite Loop:

Try → Observe → Adjust → ✅     Try → Fail → Retry same thing
  ↑              ↓                ↑                    ↓
  └── (progress) ┘                └──── (no progress) ─┘
```

## Common Causes

### 1. Same Failed Approach Repeatedly

The agent encounters an error and retries the exact same action, expecting a different result.

```
Agent: Write code → Test fails → Same code again → Test fails → Same code again...
```

### 2. Fix Oscillation

Fixing one thing breaks another, creating a cycle.

```
Agent: Fix bug A → Breaks test B → Fix test B → Breaks test A → Fix bug A...
```

### 3. Unsatisfiable Evaluator Criteria

The evaluator demands perfection the generator can't achieve.

```
Agent: Generate code → Evaluator: "needs improvement"
     → Revise code  → Evaluator: "needs improvement"
     → Revise again → Evaluator: "needs improvement" (forever)
```

### 4. Tool Returning Same Error

An external tool consistently fails, and the agent keeps calling it.

```
Agent: Call API → 500 error → Call API → 500 error → Call API...
```

### 5. The Test Modification Loop

This is the most insidious loop in agentic coding:

```
┌─────────────────────────────────────────────────────┐
│  1. Agent fails a test                              │
│  2. Agent modifies the test to pass                 │
│  3. Test "passes" but the bug remains               │
│  4. Other tests break due to actual bug             │
│  5. Agent modifies those tests too                  │
│  6. Eventually, all tests pass but code is broken   │
│                                                     │
│  Or worse:                                          │
│  4. Agent realizes modified test was wrong           │
│  5. Reverts test, original failure returns           │
│  6. Go to step 2                                    │
└─────────────────────────────────────────────────────┘
```

## Loop Detection

### Signs an Agent Is Looping

| Signal | Threshold | Action |
|--------|-----------|--------|
| Same tool call repeated | 3+ identical calls | Trigger stuck detection |
| Token spend rate | 2x expected budget | Pause and alert |
| Time without progress | 5+ minutes on same task | Force strategy change |
| Test count not improving | Same pass/fail ratio for 3 iterations | Escalate |
| File being re-edited | Same file modified 5+ times | Review approach |

### Loop Detection Pseudocode

```typescript
interface LoopDetector {
  toolCallHistory: ToolCall[];
  maxIdenticalCalls: number;
  maxIterations: number;
  budgetLimit: number;
  
  isStuck(): boolean;
  getEscapeStrategy(): Strategy;
}

function detectLoop(history: ToolCall[]): LoopState {
  // Check for identical consecutive calls
  const recent = history.slice(-5);
  const uniqueCalls = new Set(recent.map(c => `${c.tool}:${JSON.stringify(c.args)}`));
  
  if (uniqueCalls.size === 1 && recent.length >= 3) {
    return { stuck: true, reason: "identical_calls", count: recent.length };
  }
  
  // Check for oscillation (A→B→A→B pattern)
  if (history.length >= 4) {
    const last4 = history.slice(-4).map(c => c.tool);
    if (last4[0] === last4[2] && last4[1] === last4[3]) {
      return { stuck: true, reason: "oscillation", pattern: last4 };
    }
  }
  
  // Check for test count stagnation
  const testResults = history
    .filter(c => c.tool === "run_tests")
    .slice(-3)
    .map(c => c.result.passCount);
  
  if (testResults.length === 3 && new Set(testResults).size === 1) {
    return { stuck: true, reason: "no_test_progress", passCount: testResults[0] };
  }
  
  return { stuck: false };
}
```

### Practical Detection in Agent Instructions

```markdown
## Loop Prevention Rules

When you detect you are stuck:
1. If you've tried the same approach 3 times, STOP and try a completely different strategy
2. If fixing one thing breaks another, STOP and analyze the root cause before continuing
3. Never modify a test to make it pass — fix the code instead
4. If you've spent more than 5 iterations on one task, summarize what you've tried and ask for help
```

## Prevention Strategies

### 1. Max Iteration Limits

```typescript
// ✅ GOOD: Hard cap on iterations
const MAX_ITERATIONS = 10;
let iteration = 0;

while (iteration < MAX_ITERATIONS) {
  const result = await agent.step();
  
  if (result.done) break;
  iteration++;
}

if (iteration >= MAX_ITERATIONS) {
  await agent.summarizeProgress();
  await agent.requestHumanReview();
}
```

### 2. Stuck Detection with Strategy Switching

```typescript
// ✅ GOOD: Change approach when stuck
const strategies = ["direct_fix", "refactor_first", "minimal_reproduction", "ask_for_help"];
let strategyIndex = 0;
let failureCount = 0;

for (const step of agentSteps) {
  const result = await step.execute();
  
  if (!result.progress) {
    failureCount++;
    if (failureCount >= 3) {
      strategyIndex++;
      if (strategyIndex >= strategies.length) {
        return { status: "stuck", tried: strategies };
      }
      agent.setStrategy(strategies[strategyIndex]);
      failureCount = 0;
    }
  } else {
    failureCount = 0;  // Reset on progress
  }
}
```

### 3. Circuit Breakers

```typescript
// ✅ GOOD: Budget-based circuit breaker
interface CircuitBreaker {
  maxTokens: number;
  maxTime: number;
  maxToolCalls: number;
}

const breaker: CircuitBreaker = {
  maxTokens: 100_000,
  maxTime: 5 * 60 * 1000,    // 5 minutes
  maxToolCalls: 50,
};

function checkBreaker(session: AgentSession): boolean {
  if (session.tokensUsed > breaker.maxTokens) return true;
  if (Date.now() - session.startTime > breaker.maxTime) return true;
  if (session.toolCalls.length > breaker.maxToolCalls) return true;
  return false;
}
```

### 4. Immutable Tests

```markdown
## Agent Rules for Tests

- NEVER modify existing test files to make them pass
- You may ADD new test files, but not change existing ones
- If a test fails, fix the source code, not the test
- If you believe a test is wrong, flag it for human review — do not change it
```

```typescript
// ✅ GOOD: File permission enforcement
const IMMUTABLE_PATTERNS = [
  "**/*.test.ts",
  "**/*.spec.ts",
  "**/test/**",
  "**/__tests__/**",
];

function canEditFile(filepath: string): boolean {
  return !IMMUTABLE_PATTERNS.some(pattern => minimatch(filepath, pattern));
}
```

### 5. Fallback Strategies

```
Primary Approach Failed?
  ↓
├── Try alternative algorithm/pattern
│     ↓ Failed?
├── Reduce scope (fix one test at a time)
│     ↓ Failed?
├── Create minimal reproduction
│     ↓ Failed?
├── Summarize attempts and escalate to human
│     ↓
└── STOP — do not continue burning tokens
```

## Real-World Loop Examples

### ❌ The Type Error Loop

```
Iteration 1: Add type assertion → Different type error
Iteration 2: Fix new type error → Original type error returns
Iteration 3: Add type assertion → Different type error (same as #1)
Iteration 4: Fix new type error → Original type error returns
...repeats forever...
```

**Fix:** Step back and understand the type hierarchy instead of patching symptoms.

### ❌ The Import Cycle

```
Iteration 1: Move function to fix circular import → Breaks other imports
Iteration 2: Fix other imports → Creates new circular import
Iteration 3: Move function back → Original circular import
...repeats forever...
```

**Fix:** Restructure modules to break the dependency cycle, not just move code around.

## Agent Instructions for Loop Awareness

Include this in your agent configuration:

```markdown
## Self-Monitoring Rules

1. Track your attempts. Before each action, briefly note what you're trying and why
2. If your current approach matches a previous failed attempt, STOP
3. Maintain a "tried" list — never repeat an approach verbatim
4. After 3 failed attempts at the same problem:
   - Summarize what you've tried
   - Analyze WHY each attempt failed
   - Propose a fundamentally different approach
   - If no different approach exists, report the issue
5. Budget awareness: you have ~50 tool calls per task. Spend them wisely
```

## Next Steps

- [Over-Engineering](./over-engineering.md) — complexity that leads to harder-to-debug loops
- [Context Starvation](./context-starvation.md) — loops that exhaust context windows
- [Blind Delegation](./blind-delegation.md) — when loops go undetected due to lack of verification
- [Evaluator-Optimizer Pattern](../02-workflow-patterns/evaluator-optimizer.md) — building loops that actually converge
