# Error Recovery in Agentic Systems

> Handling failures gracefully — retry strategies, fallback approaches, checkpoints, rollbacks, and stopping conditions that keep agents productive when things go wrong.

---

## Common Agent Failures

Agents fail in predictable ways. Understanding these patterns lets you design recovery strategies before failures happen.

```
┌──────────────────────────────────────────────────────────────┐
│              Common Agent Failure Modes                       │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│   Tool Errors              Hallucinated Tool Calls           │
│   ───────────              ──────────────────────            │
│   API rate limits          Agent calls tool that doesn't     │
│   Network timeouts         exist or uses wrong parameters    │
│   Permission denied                                          │
│   File not found           Infinite Loops                    │
│                            ───────────────                   │
│   Context Exhaustion       Agent retries same failing        │
│   ──────────────────       approach endlessly                │
│   Context window full,                                       │
│   quality degrades         Oscillation                       │
│                            ───────────                       │
│   Stuck Detection          Fix A breaks B, fix B breaks A,   │
│   ───────────────          repeat forever                    │
│   Agent makes no progress                                    │
│   despite continued effort                                   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

## Recovery Strategy 1: Retry with Backoff

For transient errors (network, rate limits), retry with increasing delays:

```
┌──────────────────────────────────────────────────┐
│           Retry with Exponential Backoff          │
├──────────────────────────────────────────────────┤
│                                                   │
│   Attempt 1 ── Fail ── Wait 1s                    │
│   Attempt 2 ── Fail ── Wait 2s                    │
│   Attempt 3 ── Fail ── Wait 4s                    │
│   Attempt 4 ── Fail ── STOP (max retries)         │
│                                                   │
│   Key: Only retry transient errors                │
│   Don't retry: wrong file path, bad logic,        │
│   permission denied (these won't fix themselves)  │
│                                                   │
└──────────────────────────────────────────────────┘
```

```typescript
async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  maxRetries: number = 3,
  baseDelay: number = 1000
): Promise<T> {
  for (let attempt = 0; attempt <= maxRetries; attempt++) {
    try {
      return await fn();
    } catch (error) {
      if (attempt === maxRetries || !isTransient(error)) {
        throw error;
      }
      const delay = baseDelay * Math.pow(2, attempt);
      await sleep(delay);
    }
  }
  throw new Error("Unreachable");
}

function isTransient(error: unknown): boolean {
  if (error instanceof Error) {
    return (
      error.message.includes("rate limit") ||
      error.message.includes("timeout") ||
      error.message.includes("ECONNRESET")
    );
  }
  return false;
}
```

## Recovery Strategy 2: Alternative Approach

When one approach fails, try a different one:

```
┌──────────────────────────────────────────────────┐
│           Alternative Approach Strategy           │
├──────────────────────────────────────────────────┤
│                                                   │
│   Primary Approach                                │
│   ├── Success ──▶ Done                            │
│   └── Fail ──▶ Analyze why                        │
│                    │                              │
│                    ▼                              │
│              Alternative Approach                  │
│              ├── Success ──▶ Done                  │
│              └── Fail ──▶ Escalate to human        │
│                                                   │
│   Example:                                        │
│   "Add a field to the database"                   │
│   Approach 1: Prisma migration ── Fail (Prisma    │
│               config not found)                    │
│   Approach 2: Raw SQL migration ── Success         │
│                                                   │
└──────────────────────────────────────────────────┘
```

## Recovery Strategy 3: Checkpoint and Rollback

Use git as a recovery mechanism. Commit before risky operations so you can always roll back.

```
┌──────────────────────────────────────────────────────────────┐
│              Git Checkpoint Strategy                          │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│   Before Risky Operation:                                    │
│   $ git add -A && git commit -m "checkpoint: before refactor"│
│                                                              │
│   Attempt Operation:                                         │
│   ├── Success ──▶ Continue (checkpoint remains in history)   │
│   └── Fail ──▶ Rollback                                      │
│                   $ git reset --hard HEAD~1                   │
│                   Try alternative approach                    │
│                                                              │
│   Timeline:                                                  │
│   ──[checkpoint]──[risky change]──[fail]──[rollback]──       │
│                                                              │
│   Agent should commit at these points:                       │
│   • Before large refactors                                   │
│   • Before multi-file changes                                │
│   • Before modifying tests                                   │
│   • After each feature/bug fix is verified                   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Implementing Checkpoints

```bash
# Agent instruction: always checkpoint before risky operations

# Before a refactor
git add -A
git commit -m "checkpoint: before extracting UserService"

# Attempt the refactor
# ... make changes ...

# If tests fail after refactor
git reset --hard HEAD~1
# Try a different approach

# If tests pass after refactor
git add -A
git commit -m "refactor: extract UserService from monolith"
```

## Recovery Strategy 4: Human Escalation

When the agent can't recover, it should ask for help — not keep trying and making things worse.

```
┌──────────────────────────────────────────────────┐
│           Escalation Triggers                     │
├──────────────────────────────────────────────────┤
│                                                   │
│  Escalate when:                                   │
│  • Same error 3+ times with different approaches  │
│  • Change would affect more files than expected   │
│  • Test failures in unrelated areas               │
│  • Security-sensitive operation                   │
│  • Architectural decision needed                  │
│  • Conflicting requirements detected              │
│                                                   │
│  How to escalate:                                 │
│  1. Describe what was attempted                   │
│  2. Explain why it failed                         │
│  3. Suggest possible paths forward                │
│  4. Wait for human input                          │
│                                                   │
└──────────────────────────────────────────────────┘
```

## Stopping Conditions

Every agent operation needs exit conditions to prevent runaway execution.

```
┌──────────────────────────────────────────────────────────────┐
│              Stopping Conditions                              │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│   Condition              Threshold        Action             │
│   ─────────              ─────────        ──────             │
│   Max iterations         10-20 loops      Stop, report       │
│   Max time               30 minutes       Stop, checkpoint   │
│   Max cost               $5 per task      Stop, report       │
│   No progress            3 identical      Stop, escalate     │
│                          attempts                            │
│   Context usage          >90% window      Compact or stop    │
│   Test regression        New failures     Rollback, report   │
│                          introduced                          │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

## The "Stuck" Detection Pattern

An agent is stuck when it makes the same tool call multiple times with the same result:

```
┌──────────────────────────────────────────────────────────────┐
│              Stuck Detection Algorithm                        │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│   track recent_calls = []                                    │
│                                                              │
│   on each tool call:                                         │
│     recent_calls.append(call)                                │
│                                                              │
│     if last 3 calls have same tool + same input:             │
│       ──▶ STUCK: same action, same result                    │
│       ──▶ Stop and try different approach                    │
│                                                              │
│     if last 5 calls alternate between 2 patterns:            │
│       ──▶ OSCILLATING: fix A breaks B, fix B breaks A       │
│       ──▶ Stop and rethink the approach entirely             │
│                                                              │
│     if test results haven't improved in 5 iterations:        │
│       ──▶ NO PROGRESS: effort without results                │
│       ──▶ Escalate to human with findings so far             │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

```typescript
interface ToolCall {
  tool: string;
  input: string;
  output: string;
  timestamp: Date;
}

function detectStuck(calls: ToolCall[]): string | null {
  if (calls.length < 3) return null;

  const last3 = calls.slice(-3);

  // Same call repeated
  const allSame = last3.every(
    (c) => c.tool === last3[0].tool && c.input === last3[0].input
  );
  if (allSame) return "stuck: repeating same call";

  // Oscillation detection
  if (calls.length >= 6) {
    const last6 = calls.slice(-6);
    const pattern1 = `${last6[0].tool}:${last6[0].input}`;
    const pattern2 = `${last6[1].tool}:${last6[1].input}`;
    const oscillating = last6.every(
      (c, i) =>
        `${c.tool}:${c.input}` === (i % 2 === 0 ? pattern1 : pattern2)
    );
    if (oscillating) return "stuck: oscillating between two approaches";
  }

  return null;
}
```

## Error Recovery Flowchart

```
┌──────────────────────────────────────────────────────────────┐
│              Error Recovery Decision Tree                     │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│   Error Occurs                                               │
│       │                                                      │
│       ▼                                                      │
│   Is it transient? (rate limit, timeout, network)            │
│   ├── Yes ──▶ Retry with backoff (max 3 attempts)            │
│   └── No                                                     │
│       │                                                      │
│       ▼                                                      │
│   Is there an alternative approach?                          │
│   ├── Yes ──▶ Try alternative (max 2 alternatives)           │
│   └── No                                                     │
│       │                                                      │
│       ▼                                                      │
│   Can we rollback safely?                                    │
│   ├── Yes ──▶ Rollback to checkpoint, rethink                │
│   └── No                                                     │
│       │                                                      │
│       ▼                                                      │
│   Escalate to human                                          │
│   • Document what was tried                                  │
│   • Explain the failure                                      │
│   • Suggest possible paths                                   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

## Practical Tips

1. **Always commit before risky changes** — git is your safety net
2. **Set explicit iteration limits** — "try at most 5 approaches"
3. **Run tests after every change** — catch regressions early
4. **Log everything** — you'll need it for debugging
5. **Prefer rollback over patching** — a clean slate beats accumulated fixes
6. **Design for failure** — assume the agent will fail and plan the recovery path
