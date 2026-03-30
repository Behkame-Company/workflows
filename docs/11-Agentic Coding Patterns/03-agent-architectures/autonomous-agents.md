# Autonomous Agents

> Self-directed agents that plan and operate independently — returning to the human only when they need information or judgment, not permission.

---

## The Autonomous Agent Model

From Anthropic's research: autonomous agents begin with a command from (or discussion with) a human user, then **plan and operate independently**, potentially returning to the human for further information or judgment. During execution, agents gain **"ground truth" from the environment** — tool results, code execution output, test results — to assess their progress and adjust course.

```
┌──────────────────────────────────────────────────────────────────────┐
│                    AUTONOMOUS AGENT LOOP                             │
│                                                                      │
│  ┌──────────┐                                                        │
│  │  Human    │──── Initial task / goal ────┐                         │
│  │  Input    │                              │                         │
│  └──────────┘                              ▼                         │
│       ▲                          ┌──────────────────┐                │
│       │                          │   PLAN            │                │
│       │                          │   Analyze task,   │                │
│       │   Checkpoint /           │   determine steps │                │
│       │   needs info             └────────┬─────────┘                │
│       │                                   │                          │
│       │                                   ▼                          │
│       │                          ┌──────────────────┐                │
│       │                          │   ACT             │                │
│       │                          │   Execute next    │                │
│       │                          │   step using      │                │
│       │                          │   tools           │                │
│       │                          └────────┬─────────┘                │
│       │                                   │                          │
│       │                                   ▼                          │
│       │                          ┌──────────────────┐                │
│       │                          │   OBSERVE         │                │
│       │◄─── (optional) ─────────│   Ground truth    │                │
│       │                          │   from environment│                │
│       │                          └────────┬─────────┘                │
│       │                                   │                          │
│       │                                   ▼                          │
│       │                          ┌──────────────────┐    ┌────────┐ │
│       │                          │   ASSESS          │───►│  DONE  │ │
│       │                          │   Check progress  │    └────────┘ │
│       │                          │   against goal    │                │
│       │                          └────────┬─────────┘                │
│       │                                   │ not done                  │
│       │                                   └──────► back to PLAN      │
│       │                                                              │
└──────────────────────────────────────────────────────────────────────┘
```

## Why Coding Is Ideal for Autonomous Agents

Code solutions have a unique advantage: they are **verifiable through automated tests**. This gives autonomous agents a reliable ground truth signal that most other domains lack.

```
┌────────────────────────────────────────────────────────────┐
│            WHY CODE + AUTONOMOUS AGENTS WORKS              │
│                                                            │
│  ┌──────────────┐     ┌──────────────┐                    │
│  │ Write Code   │────►│ Run Tests    │                    │
│  └──────────────┘     └──────┬───────┘                    │
│         ▲                    │                             │
│         │              ┌─────┴──────┐                     │
│         │              │            │                     │
│         │           PASS          FAIL                    │
│         │              │            │                     │
│         │              ▼            ▼                     │
│         │        ┌──────────┐  ┌──────────────┐          │
│         │        │   Done   │  │ Error message │          │
│         │        └──────────┘  │ = ground truth│          │
│         │                      └──────┬───────┘          │
│         │                             │                   │
│         └─────── fix based on ────────┘                   │
│                  concrete feedback                        │
└────────────────────────────────────────────────────────────┘
```

Unlike asking an LLM "is this essay good?" (subjective), asking "do these tests pass?" gives a **binary, objective signal**. The agent can iterate using test results as concrete feedback — no human judgment needed.

## The SWE-bench Agent Pattern

The SWE-bench benchmark reveals a powerful pattern for autonomous coding agents:

```
┌────────────────────────────────────────────────────────────┐
│              SWE-BENCH AGENT PATTERN                        │
│                                                            │
│  1. READ ISSUE                                             │
│     └── Understand the bug report or feature request       │
│                                                            │
│  2. EXPLORE CODE                                           │
│     └── Search the codebase, read relevant files           │
│     └── Understand architecture and conventions            │
│                                                            │
│  3. PLAN FIX                                               │
│     └── Determine what needs to change and why             │
│                                                            │
│  4. IMPLEMENT                                              │
│     └── Make the code changes                              │
│                                                            │
│  5. TEST                                                   │
│     └── Run existing tests, write new ones if needed       │
│                                                            │
│  6. ITERATE                                                │
│     └── If tests fail, analyze errors and go back to 4     │
│     └── If tests pass, verify the fix addresses the issue  │
└────────────────────────────────────────────────────────────┘
```

### Implementation

```typescript
interface AgentState {
  goal: string;
  plan: string[];
  currentStep: number;
  history: ActionResult[];
  iteration: number;
  maxIterations: number;
}

async function autonomousAgent(task: string): Promise<AgentResult> {
  const state: AgentState = {
    goal: task,
    plan: [],
    currentStep: 0,
    history: [],
    iteration: 0,
    maxIterations: 25,  // Critical: always set a stopping condition
  };

  // Phase 1: Explore and understand
  const exploration = await llm.complete({
    system: `You are an autonomous coding agent. Explore the codebase to 
             understand the problem before making any changes.`,
    tools: [readFile, search, listDir, gitLog],
    prompt: `Task: ${task}\n\nExplore the codebase to understand what needs to change.`
  });
  state.history.push({ phase: "explore", result: exploration });

  // Phase 2: Create a plan
  const plan = await llm.complete({
    prompt: `Based on your exploration:\n${exploration}\n\n
             Create a step-by-step plan to accomplish: ${task}`
  });
  state.plan = parsePlan(plan);

  // Phase 3: Execute with iteration
  while (state.iteration < state.maxIterations) {
    state.iteration++;

    // Execute current plan step
    const action = await llm.complete({
      system: `You are implementing step ${state.currentStep + 1} of the plan.`,
      tools: [readFile, writeFile, editFile, shell],
      prompt: `Plan: ${state.plan.join("\n")}\n
               Current step: ${state.plan[state.currentStep]}\n
               History: ${JSON.stringify(state.history.slice(-5))}`
    });
    state.history.push({ phase: "act", result: action });

    // Run tests — the ground truth signal
    const testResult = await shell.execute("npm test");
    state.history.push({ phase: "test", result: testResult });

    // Assess: are we done?
    if (testResult.exitCode === 0) {
      const assessment = await llm.complete({
        prompt: `Tests pass. Does the implementation fully address the goal?
                 Goal: ${task}
                 Changes made: ${summarizeChanges(state.history)}
                 
                 Respond with DONE if complete, or describe what's still needed.`
      });

      if (assessment.includes("DONE")) {
        return { success: true, iterations: state.iteration, history: state.history };
      }
    }

    // Not done — re-plan based on current state
    const replan = await llm.complete({
      prompt: `Tests ${testResult.exitCode === 0 ? "pass" : "fail"}.
               Error output: ${testResult.stderr}
               What should we try next?`
    });
    state.plan = parsePlan(replan);
    state.currentStep = 0;
  }

  return { success: false, reason: "Max iterations reached", history: state.history };
}
```

## Stopping Conditions

Autonomous agents **must** have stopping conditions. Without them, agents can loop indefinitely, burning tokens and making increasingly questionable changes.

```typescript
// Essential stopping conditions for autonomous agents
const stoppingConditions = {
  // Hard limits
  maxIterations: 25,              // Absolute maximum loop iterations
  maxTokenBudget: 500_000,        // Total tokens consumed
  maxTimeMinutes: 30,             // Wall clock time
  
  // Soft limits (agent can choose to stop)
  maxConsecutiveFailures: 3,      // Repeated failures suggest wrong approach
  maxFilesModified: 20,           // Scope creep detection
  
  // Success conditions
  testsPass: true,                // Primary success signal
  goalAchieved: true,             // LLM assessment of completion
};

function shouldStop(state: AgentState): StopReason | null {
  if (state.iteration >= stoppingConditions.maxIterations) {
    return { reason: "max_iterations", message: "Reached iteration limit" };
  }
  
  const consecutiveFailures = countConsecutiveFailures(state.history);
  if (consecutiveFailures >= stoppingConditions.maxConsecutiveFailures) {
    return { reason: "stuck", message: "Multiple consecutive failures — may need human input" };
  }

  const filesModified = countUniqueFilesModified(state.history);
  if (filesModified > stoppingConditions.maxFilesModified) {
    return { reason: "scope_creep", message: "Too many files modified — pausing for review" };
  }

  return null;  // Continue
}
```

## Checkpoints: Human-in-the-Loop

Autonomous doesn't mean unsupervised. Good autonomous agents **pause for human feedback** at meaningful checkpoints:

```typescript
async function autonomousWithCheckpoints(task: string) {
  const checkpointTriggers = [
    "about_to_delete_files",
    "changing_public_api",
    "modifying_config",
    "plan_created",          // Let human review plan before execution
    "significant_refactor",
  ];

  // ... agent loop ...

  if (shouldCheckpoint(action, checkpointTriggers)) {
    const approval = await requestHumanFeedback({
      message: `Agent wants to: ${action.description}`,
      changes: action.proposedChanges,
      options: ["approve", "modify", "reject", "abort"],
    });

    switch (approval.decision) {
      case "approve": continue;
      case "modify": applyHumanModifications(approval.modifications); break;
      case "reject": replanWithout(action); break;
      case "abort": return { success: false, reason: "Human aborted" };
    }
  }
}
```

## When to Use Autonomous Agents

✅ **Good fit when**:
- The problem is **open-ended** — you can't predict the number of steps
- Solutions are **verifiable** — tests, type checks, linting provide ground truth
- The agent needs to **explore and discover** — not just follow instructions
- You need **trust in decision-making** — the agent must choose its own path
- Tasks are **too complex for a single prompt** but too varied for a fixed workflow

❌ **Poor fit when**:
- The task is well-defined with predictable steps (use a [workflow](../02-workflow-patterns/prompt-chaining.md))
- There's no way to verify correctness automatically
- Every step needs human approval (use [plan-and-execute](./plan-and-execute.md) instead)
- Speed matters more than thoroughness (autonomous agents are slower)

## Autonomous Agents in Practice

| Tool | Autonomous Pattern |
|------|-------------------|
| **Copilot Coding Agent** | Assigns GitHub issue → agent creates branch, explores, implements, tests, creates PR |
| **Copilot CLI** | Agent mode with full tool access — reads, writes, executes, iterates |
| **Claude Code** | Terminal agent that explores codebase, makes changes, runs tests in a loop |
| **SWE-bench agents** | Read issue → explore → plan → implement → test → iterate until solved |
