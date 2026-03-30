# Testing Agentic Systems

> How to evaluate and test agent behavior — eval suites, trajectory analysis, outcome testing, and metrics that measure whether your agents actually deliver value.

---

## The Testing Challenge

Agents are fundamentally different from traditional software:

```
┌──────────────────────────────────────────────────────────────┐
│        Traditional Software vs Agentic Systems                │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│   Traditional                  Agentic                       │
│   ───────────                  ───────                       │
│   Deterministic                Non-deterministic             │
│   Same input = same output     Same input ≠ same path        │
│   Test exact behavior          Test outcomes                 │
│   Unit testable                Trajectory testable           │
│   Fast feedback                Slow + expensive              │
│                                                              │
│   You can't test:              You can test:                 │
│   • Exact tool call sequence   • Did it achieve the goal?    │
│   • Exact code generated       • Did it take reasonable      │
│   • Exact reasoning path         steps?                      │
│                                • Did it stay within bounds?  │
│                                • Did it handle errors?       │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

## Testing Approach 1: Eval Suites

Test on known tasks with expected outcomes. The gold standard for agent evaluation.

```
┌──────────────────────────────────────────────────────────────┐
│              Eval Suite Structure                             │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│   For each eval case:                                        │
│                                                              │
│   ┌─────────────┐    ┌──────────┐    ┌──────────────┐       │
│   │   Input      │───▶│  Agent   │───▶│   Grade      │       │
│   │   (task +    │    │  runs    │    │   (did it    │       │
│   │   context)   │    │  task    │    │   succeed?)  │       │
│   └─────────────┘    └──────────┘    └──────────────┘       │
│                                                              │
│   Grading criteria:                                          │
│   • Tests pass ── does the code work?                        │
│   • No regressions ── did existing tests break?              │
│   • Meets requirements ── all acceptance criteria met?       │
│   • Within budget ── cost and time reasonable?               │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Building an Eval Case

```yaml
# eval-cases/add-email-validation.yaml
name: "Add email validation to signup form"
description: "Agent should add email validation with error display"

setup:
  branch: "main"
  files_to_seed: []  # uses current codebase

task: |
  Add email validation to the signup form at src/components/SignupForm.tsx.
  Validate on blur and on submit. Show error using the existing FormError
  component. Add tests.

success_criteria:
  - name: "Tests pass"
    check: "npm test -- --testPathPattern=SignupForm"
    expect: "exit_code_0"

  - name: "Validation rejects invalid email"
    check: "grep -r 'email.*valid' src/components/__tests__/SignupForm.test"
    expect: "match_found"

  - name: "No regressions"
    check: "npm test"
    expect: "exit_code_0"

  - name: "TypeScript compiles"
    check: "npx tsc --noEmit"
    expect: "exit_code_0"

budget:
  max_time_seconds: 300
  max_iterations: 20
```

## Testing Approach 2: Trajectory Analysis

Check whether the agent takes reasonable steps, regardless of the exact path.

```
┌──────────────────────────────────────────────────────────────┐
│              Trajectory Analysis                              │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│   Task: "Fix the login bug"                                  │
│                                                              │
│   Reasonable Trajectory:                                     │
│   1. Read the bug report            ✓ Understanding          │
│   2. Find the login code            ✓ Exploration            │
│   3. Write a failing test           ✓ Verification first     │
│   4. Fix the code                   ✓ Implementation         │
│   5. Run tests                      ✓ Verification           │
│                                                              │
│   Concerning Trajectory:                                     │
│   1. Immediately modify random file  ✗ No understanding      │
│   2. Skip tests                      ✗ No verification       │
│   3. Modify 15 files                 ✗ Shotgun approach      │
│   4. Never run tests                 ✗ No validation         │
│                                                              │
│   Trajectory checks:                                         │
│   • Did agent explore before coding?                         │
│   • Did agent run tests?                                     │
│   • Did agent modify only relevant files?                    │
│   • Did agent handle errors when they occurred?              │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Trajectory Checklist

```typescript
interface TrajectoryCheck {
  name: string;
  check: (trajectory: ToolCall[]) => boolean;
}

const trajectoryChecks: TrajectoryCheck[] = [
  {
    name: "Explored before coding",
    check: (t) => {
      const firstEdit = t.findIndex((c) => c.tool === "EditFile");
      const firstRead = t.findIndex((c) => c.tool === "ReadFile");
      return firstRead < firstEdit; // Read before edit
    },
  },
  {
    name: "Ran tests after changes",
    check: (t) => {
      const lastEdit = t.findLastIndex((c) => c.tool === "EditFile");
      const lastTest = t.findLastIndex(
        (c) => c.tool === "Shell" && c.input.includes("test")
      );
      return lastTest > lastEdit; // Tests after edits
    },
  },
  {
    name: "Didn't modify too many files",
    check: (t) => {
      const editedFiles = new Set(
        t.filter((c) => c.tool === "EditFile").map((c) => c.input)
      );
      return editedFiles.size <= 10; // Reasonable scope
    },
  },
  {
    name: "Didn't get stuck",
    check: (t) => {
      // No tool called more than 5 times with same input
      const counts = new Map<string, number>();
      for (const call of t) {
        const key = `${call.tool}:${call.input}`;
        counts.set(key, (counts.get(key) || 0) + 1);
        if (counts.get(key)! > 5) return false;
      }
      return true;
    },
  },
];
```

## Testing Approach 3: Outcome Testing

Did the agent achieve the goal? Don't care about the path — only the result.

```
┌──────────────────────────────────────────────────┐
│           Outcome Testing                         │
├──────────────────────────────────────────────────┤
│                                                   │
│   Given: A task with clear acceptance criteria    │
│   When:  Agent runs the task                      │
│   Then:  Check each criterion                     │
│                                                   │
│   Criteria types:                                 │
│   • Code compiles           (build test)          │
│   • Tests pass              (test suite)          │
│   • Feature works           (functional test)     │
│   • No regressions          (full test suite)     │
│   • Performance acceptable  (benchmark)           │
│                                                   │
│   Grade: pass/fail per criterion                  │
│   Score: % of criteria met                        │
│                                                   │
└──────────────────────────────────────────────────┘
```

## Testing Approach 4: Regression Testing

Agents should solve previously solved tasks. Track a "golden set" of tasks the agent has solved before.

```
┌──────────────────────────────────────────────────────────────┐
│              Regression Testing                               │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│   Golden Set:                                                │
│   ┌─────────────────────────────────────────┐                │
│   │ Task 1: Add email validation    ✓ Pass  │                │
│   │ Task 2: Fix login bug           ✓ Pass  │                │
│   │ Task 3: Add pagination          ✓ Pass  │                │
│   │ Task 4: Refactor UserService    ✓ Pass  │                │
│   │ Task 5: Add webhook support     ✗ Fail  │ ← Regression  │
│   └─────────────────────────────────────────┘                │
│                                                              │
│   Run golden set:                                            │
│   • After changing agent instructions                        │
│   • After changing model version                             │
│   • After changing tools/skills                              │
│   • Periodically (weekly) to catch model drift               │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

## The SWE-bench Approach

SWE-bench tests agents on real GitHub issues with real codebases:

```
1. Take a real issue from a real repository
2. Check out the codebase at the time of the issue
3. Give the agent the issue description
4. Agent implements a fix
5. Run the repository's test suite
6. Grade: did the fix work? Did tests pass?
```

### Building Your Own SWE-bench

Collect tasks from your team's actual work:

```markdown
## Internal Eval Suite

### Sources for eval tasks
1. Closed PRs with clear descriptions and tests
2. Bug reports that were successfully fixed
3. Feature requests that were implemented
4. Refactoring tasks with before/after tests

### For each task, capture:
- The initial state (git commit before the fix)
- The task description (issue or PR description)
- The expected outcome (tests that should pass)
- The actual solution (for reference, not for grading)
```

## Key Metrics

```
┌──────────────────────────────────────────────────────────────┐
│              Agent Evaluation Metrics                         │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│   Metric                  What It Measures                   │
│   ──────                  ────────────────                   │
│   Success Rate            % of tasks completed successfully  │
│   Cost per Task           Average tokens/dollars per task    │
│   Time per Task           Average wall-clock time            │
│   Human Intervention      % of tasks needing human help      │
│   Rate                                                       │
│   Regression Rate         % of previously solved tasks       │
│                           that now fail                      │
│   Trajectory Quality      % of tasks with reasonable paths   │
│                                                              │
│   Good benchmarks:                                           │
│   • Success rate > 80% on well-scoped tasks                  │
│   • Human intervention < 20%                                 │
│   • Zero regression rate                                     │
│   • Cost < 2x human time equivalent                          │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

## Eval Suite Template

```yaml
# eval-suite.yaml
name: "Project Agent Eval Suite"
version: "1.0"
model: "claude-sonnet-4-20250514"

defaults:
  max_time_seconds: 600
  max_cost_dollars: 5.00
  require_tests_pass: true

cases:
  - id: "email-validation"
    task: "Add email validation to SignupForm"
    setup_commit: "abc123"
    success_checks:
      - "npm test -- --testPathPattern=SignupForm"
      - "npx tsc --noEmit"
      - "npm test"

  - id: "fix-login-bug"
    task: "Fix: users with + in email can't log in"
    setup_commit: "def456"
    success_checks:
      - "npm test -- --testPathPattern=auth"
      - "npm test"

  - id: "add-pagination"
    task: "Add cursor-based pagination to GET /api/products"
    setup_commit: "ghi789"
    success_checks:
      - "npm test -- --testPathPattern=products"
      - "npm test"
      - "npx tsc --noEmit"

reporting:
  output: "eval-results.json"
  metrics:
    - success_rate
    - average_time
    - average_cost
    - regression_count
```
