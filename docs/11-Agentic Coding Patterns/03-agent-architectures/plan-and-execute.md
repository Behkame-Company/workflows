# Plan-and-Execute Pattern

> Create an explicit plan first, then execute steps — separating the thinking phase from the doing phase for more predictable, reviewable agent behavior.

---

## The Pattern

Unlike [ReAct](./react-pattern.md) where reasoning and acting are interleaved on every step, plan-and-execute **separates planning from execution**. The agent first analyzes the task and creates a complete plan, then executes each step, optionally re-planning when reality diverges from expectations.

```
┌──────────────────────────────────────────────────────────────────┐
│                   PLAN-AND-EXECUTE                                │
│                                                                  │
│  ┌──────────────────────────────────────────────┐               │
│  │              PLANNING PHASE                   │               │
│  │                                               │               │
│  │  1. Analyze the task                          │               │
│  │  2. Explore codebase for context              │               │
│  │  3. Identify what needs to change             │               │
│  │  4. Create ordered list of steps              │               │
│  │  5. Present plan for review (optional)        │               │
│  └─────────────────────┬────────────────────────┘               │
│                        │                                         │
│                        ▼                                         │
│  ┌──────────────────────────────────────────────┐               │
│  │             EXECUTION PHASE                   │               │
│  │                                               │               │
│  │  For each step in plan:                       │               │
│  │    ├── Execute the step                       │               │
│  │    ├── Verify step succeeded                  │               │
│  │    └── If deviation detected → RE-PLAN        │               │
│  └─────────────────────┬────────────────────────┘               │
│                        │                                         │
│                        ▼                                         │
│  ┌──────────────────────────────────────────────┐               │
│  │             VERIFICATION                      │               │
│  │                                               │               │
│  │  Run tests, verify all steps completed,       │               │
│  │  confirm goal achieved                        │               │
│  └──────────────────────────────────────────────┘               │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

## Why Separate Planning from Execution?

```
┌────────────────────────────────────────────────────────────────┐
│  ReAct (interleaved)           Plan-and-Execute (separated)    │
│                                                                │
│  Think → Act → Think → Act     Plan → [Step1, Step2, Step3]   │
│       → Think → Act                  → Execute Step1          │
│       → Think → Act                  → Execute Step2          │
│       ...                            → Execute Step3          │
│                                      → Verify                 │
│                                                                │
│  ✓ Adaptive                    ✓ Predictable                  │
│  ✓ Good for exploration        ✓ Reviewable by humans         │
│  ✗ Hard to predict path        ✓ Catches misunderstandings    │
│  ✗ Can wander                  ✓ Scope is clear upfront       │
│                                ✗ Plan may not survive reality  │
└────────────────────────────────────────────────────────────────┘
```

Key benefits:
1. **Human review** — plans can be approved before execution begins
2. **Predictability** — you know what will happen before it happens
3. **Scope control** — the plan defines boundaries; prevents drift
4. **Misunderstanding detection** — wrong assumptions surface in the plan, not in broken code

## Plan-and-Execute in Copilot CLI

Copilot CLI implements plan-and-execute through **plan mode**:

- **Shift+Tab** to enter plan mode (or use the `/plan` command)
- Copilot analyzes the request and asks clarifying questions
- Builds a plan before writing any code
- Human reviews and approves before execution begins

```
┌────────────────────────────────────────────────────────────────┐
│            COPILOT CLI PLAN MODE                                │
│                                                                │
│  User: "Add rate limiting to the API"                          │
│                                                                │
│  [Shift+Tab → Plan Mode]                                       │
│                                                                │
│  Copilot: "Before implementing, let me clarify:                │
│    1. Which endpoints need rate limiting?                       │
│    2. What limits? (requests per minute/hour)                   │
│    3. Per-user or per-IP?                                       │
│    4. Should I use an existing library like express-rate-limit?" │
│                                                                │
│  User answers questions...                                      │
│                                                                │
│  Copilot: "Here's my plan:                                     │
│    1. Install express-rate-limit                                │
│    2. Create rate limit middleware in src/middleware/            │
│    3. Apply to /api/* routes with 100 req/min per IP            │
│    4. Add rate limit headers to responses                       │
│    5. Add tests for rate limiting behavior                      │
│    6. Update API docs"                                          │
│                                                                │
│  User: "Looks good, go ahead"                                   │
│  → Copilot executes the plan                                   │
└────────────────────────────────────────────────────────────────┘
```

This catches misunderstandings **before code is written** — far cheaper than fixing wrong code.

## Implementation

### The Planner

```typescript
interface Plan {
  goal: string;
  analysis: string;
  steps: PlanStep[];
  risks: string[];
  estimatedFiles: string[];
}

interface PlanStep {
  id: number;
  description: string;
  files: string[];
  dependencies: number[];  // IDs of steps that must complete first
  verification: string;     // How to verify this step succeeded
}

async function createPlan(task: string): Promise<Plan> {
  // Phase 1: Explore and understand
  const exploration = await llm.complete({
    system: `You are a senior engineer planning a code change. 
             Explore the codebase thoroughly before creating a plan.
             Understand the architecture, conventions, and existing patterns.`,
    tools: [readFile, search, listDir, gitLog],
    prompt: `Task: ${task}\n\nExplore the codebase to understand what needs to change.`
  });

  // Phase 2: Create the plan
  const plan = await llm.complete({
    system: `You are a technical architect. Create a detailed, step-by-step plan.
             
             Each step must:
             - Be specific enough to execute without ambiguity
             - List exact files to modify or create
             - Include a verification check
             - Note dependencies on other steps
             
             Also identify risks and potential issues.`,
    prompt: `Based on this exploration:
             ${exploration}
             
             Create a plan to: ${task}
             
             Return as JSON: {
               "goal": "...",
               "analysis": "...",
               "steps": [{"id": 1, "description": "...", "files": [...], 
                          "dependencies": [], "verification": "..."}],
               "risks": ["..."],
               "estimatedFiles": ["..."]
             }`
  });

  return JSON.parse(plan);
}
```

### The Executor

```typescript
async function executePlan(plan: Plan): Promise<ExecutionResult> {
  const results: StepResult[] = [];
  
  for (const step of topologicalSort(plan.steps)) {
    console.log(`\nExecuting step ${step.id}: ${step.description}`);
    
    // Check dependencies
    const depsFailed = step.dependencies.some(
      depId => results.find(r => r.stepId === depId)?.success === false
    );
    if (depsFailed) {
      console.log(`  Skipping — dependency failed`);
      results.push({ stepId: step.id, success: false, reason: "Dependency failed" });
      continue;
    }

    // Execute the step
    const result = await llm.complete({
      system: `You are executing one step of a plan. Focus only on this step.
               Do not modify files outside your scope.
               
               Overall goal: ${plan.goal}
               Your step: ${step.description}
               Files to work with: ${step.files.join(", ")}`,
      tools: [readFile, writeFile, editFile, shell],
      prompt: `Execute this step: ${step.description}
               
               Context from previous steps:
               ${results.filter(r => r.success).map(r => r.summary).join("\n")}`
    });

    // Verify the step
    const verification = await verifyStep(step, result);
    
    if (!verification.passed) {
      console.log(`  Step failed verification: ${verification.reason}`);
      
      // Decide: retry, re-plan, or continue
      const decision = await handleStepFailure(step, verification, plan);
      if (decision === "replan") {
        return await replanAndExecute(plan, results, step, verification);
      }
    }

    results.push({
      stepId: step.id,
      success: verification.passed,
      summary: result,
      verification
    });
  }

  return { plan, results, overallSuccess: results.every(r => r.success) };
}
```

### The Re-Planner

When reality diverges from the plan, re-plan rather than blindly continuing. The executor detects step failure, collects context (what completed, what failed, why), and asks the planner to create a revised plan for the remaining work.

```typescript
async function replanAndExecute(
  originalPlan: Plan,
  completedResults: StepResult[],
  failedStep: PlanStep,
  failure: VerificationResult
): Promise<ExecutionResult> {
  const revisedPlan = await llm.complete({
    system: "Adjust the remaining plan based on what actually happened.",
    prompt: `Original: ${JSON.stringify(originalPlan)}
             Completed: ${JSON.stringify(completedResults)}
             Failed: ${failedStep.description} — ${failure.reason}
             Create a revised plan for remaining work.`
  });
  return await executePlan(JSON.parse(revisedPlan));
}
```

For error recovery patterns including retry strategies, loop detection, and escalation, see [Infinite Agent Loops](../10-anti-patterns/infinite-loops.md).
```

### Python Implementation

```python
from dataclasses import dataclass, field

@dataclass
class PlanStep:
    id: int
    description: str
    files: list[str]
    dependencies: list[int] = field(default_factory=list)
    verification: str = ""

@dataclass
class Plan:
    goal: str
    steps: list[PlanStep]
    risks: list[str] = field(default_factory=list)

def plan_and_execute(task: str) -> dict:
    # Phase 1: Create the plan
    plan = create_plan(task)
    print(f"Plan created with {len(plan.steps)} steps:")
    for step in plan.steps:
        print(f"  {step.id}. {step.description}")

    # Phase 2: Optional human review
    if requires_approval(plan):
        approval = get_human_approval(plan)
        if not approval.approved:
            plan = revise_plan(plan, approval.feedback)

    # Phase 3: Execute
    results = []
    for step in topological_sort(plan.steps):
        print(f"\nExecuting: {step.description}")
        result = execute_step(step, context=results)

        if not result.success:
            # Re-plan from this point
            remaining = [s for s in plan.steps if s.id > step.id]
            new_steps = replan(plan.goal, results, step, result.error)
            plan.steps = [s for s in plan.steps if s.id <= step.id] + new_steps
            continue

        results.append(result)

    # Phase 4: Final verification
    return verify_all(plan, results)
```

## The Multi-Window Pattern

From Claude Code's approach: use multiple windows where an **initializer agent creates the plan** and **worker agents execute individual steps**.

```
┌────────────────────────────────────────────────────────────────┐
│                MULTI-WINDOW PLAN-AND-EXECUTE                    │
│                                                                │
│  ┌─────────────────────┐                                      │
│  │  INITIALIZER WINDOW  │                                      │
│  │                       │                                      │
│  │  • Analyzes task      │                                      │
│  │  • Explores codebase  │                                      │
│  │  • Creates interfaces │                                      │
│  │  • Writes plan.md     │──── plan.md ────┐                   │
│  └─────────────────────┘                  │                   │
│                                            │                   │
│              ┌─────────────────────────────┘                   │
│              │                                                  │
│              ▼                                                  │
│  ┌───────────────┐  ┌───────────────┐  ┌───────────────┐      │
│  │  WORKER WIN 1  │  │  WORKER WIN 2  │  │  WORKER WIN 3  │      │
│  │                │  │                │  │                │      │
│  │  Reads plan    │  │  Reads plan    │  │  Reads plan    │      │
│  │  Implements    │  │  Implements    │  │  Implements    │      │
│  │  step 1        │  │  step 2        │  │  step 3        │      │
│  └───────────────┘  └───────────────┘  └───────────────┘      │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

```typescript
// Copilot CLI equivalent using sub-agents
async function multiAgentPlanExecute(task: string) {
  // Initializer: create plan and interfaces
  const plan = await subAgent({
    type: "general-purpose",
    prompt: `Analyze this task and create a detailed plan with interfaces.
             Write the plan to .planning/plan.md
             Write interfaces to src/interfaces/
             Task: ${task}`
  });

  // Workers: execute plan steps in parallel where possible
  const independentSteps = findIndependentSteps(plan);
  
  await Promise.all(independentSteps.map(step =>
    subAgent({
      type: "task",
      prompt: `Read the plan at .planning/plan.md
               Execute step ${step.id}: ${step.description}
               Follow the interfaces defined in src/interfaces/`
    })
  ));
}
```

## When to Prefer Plan-and-Execute Over ReAct

| Situation | Use Plan-and-Execute | Use ReAct |
|-----------|---------------------|-----------|
| Complex multi-step task | ✅ Plan gives structure | May wander |
| Need human approval | ✅ Review plan first | Can't review emergent path |
| Predictability required | ✅ Steps are known upfront | Path varies each run |
| Exploratory debugging | May over-plan | ✅ Discover as you go |
| Simple one-shot fix | Over-engineering | ✅ Think, fix, verify |
| Team coordination | ✅ Plan is a shared artifact | Hard to coordinate |

## Design Guidelines

1. **Plans should be specific** — "Update the auth middleware" is too vague; "Add JWT validation to `src/middleware/auth.ts` checking the `Authorization` header" is actionable
2. **Include verification for each step** — how will you know the step succeeded?
3. **Budget for re-planning** — no plan survives first contact; build in replanning triggers
4. **Don't over-plan** — 3-7 steps is ideal; more than 10 suggests the task should be decomposed
5. **Plans are living documents** — update the plan as reality emerges during execution
