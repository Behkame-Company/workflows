# Orchestrator-Workers

> A central LLM dynamically breaks down tasks, delegates to specialized workers, and synthesizes results — the pattern behind modern coding agents.

---

## The Pattern

The orchestrator-workers pattern uses a **central LLM (the orchestrator)** to analyze a task, dynamically determine what subtasks are needed, delegate each to **specialized workers**, and then synthesize the results.

```
┌──────────────────────────────────────────────────────────────────┐
│                    ORCHESTRATOR-WORKERS                           │
│                                                                  │
│                    ┌────────────────┐                            │
│                    │  ORCHESTRATOR  │                            │
│                    │                │                            │
│                    │ • Analyzes task│                            │
│                    │ • Plans work   │                            │
│                    │ • Delegates    │                            │
│                    │ • Synthesizes  │                            │
│                    └───────┬────────┘                            │
│                            │                                     │
│              ┌─────────────┼─────────────┐                      │
│              │ dynamically  │ determined  │                      │
│              ▼             ▼             ▼                      │
│        ┌──────────┐ ┌──────────┐ ┌──────────┐                  │
│        │ Worker 1 │ │ Worker 2 │ │ Worker N │                  │
│        │          │ │          │ │          │   Number and      │
│        │ Edit     │ │ Create   │ │ Update   │   nature of      │
│        │ auth.ts  │ │ jwt.ts   │ │ tests    │   workers is     │
│        └────┬─────┘ └────┬─────┘ └────┬─────┘   determined     │
│             │            │            │          at runtime     │
│             └────────────┼────────────┘                         │
│                          ▼                                      │
│                   ┌──────────────┐                              │
│                   │  SYNTHESIZE  │                              │
│                   │  Combine all │                              │
│                   │  worker      │                              │
│                   │  results     │                              │
│                   └──────────────┘                              │
└──────────────────────────────────────────────────────────────────┘
```

**Key difference from [parallelization](./parallelization.md)**: In parallelization, subtasks are predefined by the developer. In orchestrator-workers, the **orchestrator dynamically determines** what subtasks are needed based on the input. You can't know in advance how many workers will be spawned or what they'll do.

## When to Use Orchestrator-Workers

✅ **Good fit when**:
- **Subtasks can't be predicted in advance** — the number and nature of work items depends on the input
- The task requires **analysis before delegation** — you need to understand the problem to break it down
- Workers can operate **relatively independently** once given their assignment

❌ **Poor fit when**:
- Subtasks are always the same (use fixed [parallelization](./parallelization.md) or [prompt chaining](./prompt-chaining.md))
- The task is simple enough for a single call
- Workers need heavy back-and-forth coordination (consider a single agent loop instead)

## This Is How Coding Agents Work

Modern coding agents — including GitHub Copilot CLI with its sub-agent architecture — follow the orchestrator-workers pattern:

```
┌────────────────────────────────────────────────────────────┐
│              CODING AGENT AS ORCHESTRATOR                   │
│                                                            │
│  User: "Add authentication to the API"                     │
│                                                            │
│  Orchestrator analyzes:                                    │
│  ├── Need to create auth middleware (→ Worker 1)           │
│  ├── Need to add JWT utilities (→ Worker 2)                │
│  ├── Need to update 4 route files (→ Workers 3-6)         │
│  ├── Need to add user model (→ Worker 7)                   │
│  ├── Need to update config (→ Worker 8)                    │
│  └── Need to add tests (→ Worker 9)                        │
│                                                            │
│  The orchestrator couldn't know it was 9 workers           │
│  until it analyzed the specific codebase                   │
└────────────────────────────────────────────────────────────┘
```

GitHub Copilot CLI's task tool with agent types like `explore`, `task`, and `general-purpose` is a direct implementation of this pattern — the main agent (orchestrator) spawns specialized sub-agents (workers) based on what the task requires.

## Implementation: Dynamic Task Decomposition

```typescript
interface WorkerTask {
  id: string;
  description: string;
  files: string[];
  dependencies: string[];  // IDs of tasks that must complete first
}

interface WorkerResult {
  taskId: string;
  success: boolean;
  changes: FileChange[];
  error?: string;
}

// The Orchestrator: analyze, plan, delegate, synthesize
async function orchestrate(userTask: string) {
  // Phase 1: ANALYZE — understand the codebase and task
  const analysis = await llm.complete({
    system: `You are a senior software architect. Analyze the task and codebase to 
             determine exactly what needs to change. Be thorough — missing a file 
             means a worker won't be assigned to it.`,
    tools: [readFile, search, listDirectory],
    prompt: `Task: ${userTask}
             
             Explore the codebase to understand:
             1. What files exist and what they do
             2. What needs to change to accomplish this task
             3. What new files need to be created
             4. What dependencies exist between changes`
  });

  // Phase 2: PLAN — decompose into worker tasks
  const plan = await llm.complete({
    system: `You are a technical project manager. Break down the work into independent 
             tasks that can be assigned to individual developers (workers).
             
             Each task should:
             - Be completable independently
             - Touch a small, focused set of files  
             - Have clear success criteria
             - List dependencies on other tasks`,
    prompt: `Based on this analysis, create a task breakdown:
             ${analysis}
             
             Return JSON array of tasks:
             [{ "id": "task-1", "description": "...", "files": [...], "dependencies": [...] }]`
  });

  const tasks: WorkerTask[] = JSON.parse(plan);
  console.log(`Orchestrator created ${tasks.length} worker tasks`);

  // Phase 3: DELEGATE — execute tasks respecting dependencies
  const results = await executeWithDependencies(tasks);

  // Phase 4: SYNTHESIZE — combine and verify results
  const synthesis = await synthesizeResults(userTask, results);
  return synthesis;
}
```

### Executing Workers with Dependency Ordering

```typescript
async function executeWithDependencies(tasks: WorkerTask[]): Promise<WorkerResult[]> {
  const completed = new Map<string, WorkerResult>();
  const pending = new Set(tasks.map(t => t.id));

  while (pending.size > 0) {
    // Find tasks whose dependencies are all completed
    const ready = tasks.filter(t => 
      pending.has(t.id) && 
      t.dependencies.every(dep => completed.has(dep))
    );

    if (ready.length === 0 && pending.size > 0) {
      throw new Error("Circular dependency detected in task graph");
    }

    // Execute all ready tasks in parallel
    const results = await Promise.all(
      ready.map(task => executeWorker(task, completed))
    );

    // Record completions
    for (const result of results) {
      completed.set(result.taskId, result);
      pending.delete(result.taskId);
    }

    console.log(`Completed ${ready.length} tasks. ${pending.size} remaining.`);
  }

  return Array.from(completed.values());
}
```

### The Worker: Focused Execution

```typescript
async function executeWorker(
  task: WorkerTask,
  priorResults: Map<string, WorkerResult>
): Promise<WorkerResult> {
  // Gather context from prior tasks this one depends on
  const dependencyContext = task.dependencies
    .map(depId => {
      const result = priorResults.get(depId);
      return `Task "${depId}" completed: ${JSON.stringify(result?.changes)}`;
    })
    .join("\n");

  try {
    const result = await llm.complete({
      system: `You are a focused developer working on a single task. 
               Complete ONLY your assigned task. Do not modify files outside your scope.
               
               Your task: ${task.description}
               Your files: ${task.files.join(", ")}`,
      tools: [readFile, writeFile, editFile, runTests],
      prompt: `Complete this task:
               ${task.description}
               
               Files to work with: ${task.files.join(", ")}
               
               ${dependencyContext ? `Context from prior tasks:\n${dependencyContext}` : ""}`
    });

    return {
      taskId: task.id,
      success: true,
      changes: extractFileChanges(result)
    };
  } catch (error) {
    return {
      taskId: task.id,
      success: false,
      changes: [],
      error: String(error)
    };
  }
}
```

### Synthesis: Combining Worker Results

```typescript
async function synthesizeResults(
  originalTask: string,
  results: WorkerResult[]
): Promise<SynthesisResult> {
  const failures = results.filter(r => !r.success);
  const successes = results.filter(r => r.success);

  if (failures.length > 0) {
    console.warn(`${failures.length} workers failed:`, 
      failures.map(f => `${f.taskId}: ${f.error}`));
  }

  // Verify all changes work together
  const verification = await llm.complete({
    system: `You are a senior engineer reviewing work from multiple developers.
             Verify that all changes are consistent and work together correctly.
             Check for: import consistency, API contract matches, no conflicting changes.`,
    tools: [readFile, search, runTests],
    prompt: `Original task: ${originalTask}
             
             Changes made by ${successes.length} workers:
             ${successes.map(s => 
               `Worker ${s.taskId}: ${JSON.stringify(s.changes)}`
             ).join("\n\n")}
             
             Run tests and verify everything works together.`
  });

  return { verification, results, allPassed: failures.length === 0 };
}
```

## Orchestrator Design Patterns

### Pattern 1: Exploratory Orchestrator

The orchestrator explores the codebase before planning:

```typescript
async function exploratoryOrchestrator(task: string) {
  // Explore first — orchestrator has tools to understand the codebase
  const understanding = await llm.complete({
    system: "Explore the codebase to understand its structure before planning any changes.",
    tools: [readFile, search, listDirectory, gitLog],
    prompt: task
  });

  // Then plan based on actual codebase understanding
  const tasks = await planTasks(understanding);
  
  // Then execute
  return await executeWithDependencies(tasks);
}
```

### Pattern 2: Iterative Orchestrator

The orchestrator can spawn additional workers based on results:

```typescript
async function iterativeOrchestrator(task: string) {
  let allResults: WorkerResult[] = [];
  let remainingWork = task;

  for (let iteration = 0; iteration < 3; iteration++) {
    const tasks = await planTasks(remainingWork);
    if (tasks.length === 0) break;  // No more work needed

    const results = await executeWithDependencies(tasks);
    allResults.push(...results);

    // Check if more work is needed
    const assessment = await llm.complete({
      prompt: `Original task: ${task}\nCompleted: ${JSON.stringify(allResults)}\nIs more work needed?`
    });

    if (assessment.includes("complete")) break;
    remainingWork = assessment;  // Orchestrator identifies remaining work
  }

  return allResults;
}
```

## Design Guidelines

1. **Orchestrator should analyze before delegating** — don't rush to spawn workers without understanding the problem
2. **Workers should be focused** — each worker gets a small, clear scope
3. **Workers should be stateless** — provide all context the worker needs upfront; don't assume shared state
4. **Handle partial failure gracefully** — some workers may fail; the orchestrator decides how to proceed
5. **Verify after synthesis** — run tests and checks after combining worker results
6. **Limit worker count** — too many workers increases coordination cost and error probability

## Next Steps

- [Evaluator-Optimizer](./evaluator-optimizer.md) — add quality loops to orchestrated work
- [Parallelization](./parallelization.md) — for when subtasks are known in advance
- [When to Go Agentic](../01-fundamentals/when-to-go-agentic.md) — deciding between orchestrator and simpler patterns
