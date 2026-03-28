# Multi-Agent Patterns

> Four primary architectures for systems where multiple AI agents collaborate — coordinator, pipeline, debate, and swarm — each with distinct tradeoffs for coding tasks.

---

## Why Multi-Agent?

A single agent works well for focused tasks. But complex coding work — large features, codebase-wide refactors, thorough code review — benefits from **multiple agents with different roles or perspectives**. Multi-agent systems decompose work in ways that improve quality, speed, or both.

```
┌──────────────────────────────────────────────────────────────────┐
│              FOUR MULTI-AGENT PATTERNS                            │
│                                                                  │
│  ┌────────────────┐    ┌────────────────┐                       │
│  │  COORDINATOR   │    │   PIPELINE     │                       │
│  │                │    │                │                       │
│  │  Central agent │    │  Sequential    │                       │
│  │  delegates to  │    │  handoff from  │                       │
│  │  specialists   │    │  agent to agent│                       │
│  └────────────────┘    └────────────────┘                       │
│                                                                  │
│  ┌────────────────┐    ┌────────────────┐                       │
│  │    DEBATE      │    │    SWARM       │                       │
│  │                │    │                │                       │
│  │  Agents argue  │    │  Peer agents   │                       │
│  │  for different │    │  work on tasks │                       │
│  │  approaches    │    │  independently │                       │
│  └────────────────┘    └────────────────┘                       │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

## Pattern 1: Coordinator

A **central agent** receives the task, analyzes it, delegates subtasks to specialist agents, and synthesizes results.

```
┌────────────────────────────────────────────────────────────────┐
│                      COORDINATOR                                │
│                                                                │
│                  ┌──────────────┐                              │
│                  │ COORDINATOR  │                              │
│                  │              │                              │
│                  │ Analyzes     │                              │
│                  │ Delegates    │                              │
│                  │ Synthesizes  │                              │
│                  └──────┬───────┘                              │
│                         │                                      │
│           ┌─────────────┼─────────────┐                       │
│           ▼             ▼             ▼                       │
│     ┌──────────┐  ┌──────────┐  ┌──────────┐                 │
│     │ Frontend │  │ Backend  │  │  Testing  │                 │
│     │ Agent    │  │ Agent    │  │  Agent    │                 │
│     └──────────┘  └──────────┘  └──────────┘                 │
│           │             │             │                       │
│           └─────────────┼─────────────┘                       │
│                         ▼                                      │
│                  ┌──────────────┐                              │
│                  │  SYNTHESIZE  │                              │
│                  │  Merge +     │                              │
│                  │  Verify      │                              │
│                  └──────────────┘                              │
│                                                                │
│  When to use:                                                  │
│  • Complex features spanning multiple domains                  │
│  • Agent specialization improves quality                       │
│  • Dynamic — coordinator decides how many agents needed        │
└────────────────────────────────────────────────────────────────┘
```

```typescript
// Coordinator pattern implementation
async function coordinatorPattern(task: string) {
  // Coordinator analyzes and delegates
  const analysis = await coordinator.analyze(task);
  
  const subtasks = analysis.decompose();
  // [
  //   { domain: "frontend", task: "Build login form component" },
  //   { domain: "backend",  task: "Create auth API endpoint" },
  //   { domain: "testing",  task: "Write integration tests for auth" },
  // ]

  // Delegate to specialist agents
  const results = await Promise.all(
    subtasks.map(sub => specialists[sub.domain].execute(sub.task))
  );

  // Coordinator synthesizes and verifies
  return await coordinator.synthesize(results);
}
```

## Pattern 2: Pipeline

Agents process work **sequentially**, each adding their specialty before handing off to the next.

```
┌────────────────────────────────────────────────────────────────┐
│                       PIPELINE                                  │
│                                                                │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐  │
│  │ PLANNER  │──►│  CODER   │──►│ REVIEWER │──►│ TESTER   │  │
│  │          │   │          │   │          │   │          │  │
│  │ Analyzes │   │ Writes   │   │ Reviews  │   │ Writes & │  │
│  │ task,    │   │ code     │   │ for bugs │   │ runs     │  │
│  │ creates  │   │ based on │   │ security │   │ tests    │  │
│  │ spec     │   │ spec     │   │ style    │   │          │  │
│  └──────────┘   └──────────┘   └──────────┘   └──────────┘  │
│       │              │              │              │           │
│       ▼              ▼              ▼              ▼           │
│    spec.md        code files    review.md     test results    │
│                                                                │
│  When to use:                                                  │
│  • Well-defined stages (like code review workflows)            │
│  • Each stage adds distinct value                              │
│  • Output of one stage feeds next                              │
└────────────────────────────────────────────────────────────────┘
```

```typescript
// Pipeline pattern implementation
async function pipelinePattern(task: string) {
  // Stage 1: Plan
  const spec = await plannerAgent.createSpec(task);
  
  // Stage 2: Implement
  const code = await coderAgent.implement(spec);
  
  // Stage 3: Review
  const review = await reviewerAgent.review(code, spec);
  
  // If review finds issues, loop back to coder
  if (review.hasIssues) {
    const fixedCode = await coderAgent.fix(code, review.issues);
    const reReview = await reviewerAgent.review(fixedCode, spec);
    // ... until review passes
  }
  
  // Stage 4: Test
  const tests = await testerAgent.writeAndRun(code, spec);
  
  return { code, review, tests };
}
```

## Pattern 3: Debate

Multiple agents **argue for different approaches**, and a judge selects the best solution.

```
┌────────────────────────────────────────────────────────────────┐
│                        DEBATE                                   │
│                                                                │
│  Task: "Design the caching strategy for the API"               │
│                                                                │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐        │
│  │  AGENT A      │  │  AGENT B      │  │  AGENT C      │        │
│  │               │  │               │  │               │        │
│  │  "Use Redis   │  │  "Use in-     │  │  "Use CDN     │        │
│  │   with TTL    │  │   memory cache│  │   edge caching│        │
│  │   invalidation│  │   (simpler,   │  │   (fastest for│        │
│  │   for data    │  │   fewer deps, │  │   read-heavy  │        │
│  │   consistency"│  │   good enough │  │   workloads)" │        │
│  │               │  │   for now)"   │  │               │        │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘        │
│         │                  │                  │                 │
│         └──────────────────┼──────────────────┘                 │
│                            ▼                                    │
│                   ┌──────────────┐                              │
│                   │    JUDGE     │                              │
│                   │              │                              │
│                   │  Evaluates   │                              │
│                   │  arguments,  │                              │
│                   │  picks best  │                              │
│                   │  approach or │                              │
│                   │  synthesizes │                              │
│                   └──────────────┘                              │
│                                                                │
│  When to use:                                                  │
│  • Architecture decisions with multiple valid approaches       │
│  • Want diverse perspectives                                   │
│  • Quality from competition between agents                     │
└────────────────────────────────────────────────────────────────┘
```

```typescript
// Debate pattern implementation
async function debatePattern(task: string, perspectives: string[]) {
  // Generate multiple approaches in parallel
  const proposals = await Promise.all(
    perspectives.map(perspective =>
      llm.complete({
        system: `You are an expert advocating for: ${perspective}.
                 Make the strongest case for your approach.
                 Include: rationale, implementation plan, tradeoffs, risks.`,
        prompt: task
      })
    )
  );

  // Optional: let agents critique each other
  const critiques = await Promise.all(
    proposals.map((proposal, i) =>
      llm.complete({
        system: `You are reviewing a proposed approach. Be constructively critical.
                 Point out weaknesses, missing considerations, and risks.`,
        prompt: `Proposal: ${proposal}\n\nOther proposals:\n${
          proposals.filter((_, j) => j !== i).join("\n---\n")
        }`
      })
    )
  );

  // Judge evaluates all proposals and critiques
  const decision = await llm.complete({
    system: `You are a senior architect making a final decision.
             Evaluate all proposals and critiques objectively.
             You may pick one approach or synthesize the best elements of each.`,
    prompt: `Proposals and critiques:\n${
      proposals.map((p, i) => `## Approach ${i+1}\n${p}\n\nCritique:\n${critiques[i]}`).join("\n\n")
    }`
  });

  return decision;
}
```

## Pattern 4: Swarm

**Peer agents** work independently on parallel subtasks with minimal coordination.

```
┌────────────────────────────────────────────────────────────────┐
│                        SWARM                                    │
│                                                                │
│  Task: "Migrate all 12 API endpoints from v1 to v2"           │
│                                                                │
│  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐     │
│  │Agent 1 │ │Agent 2 │ │Agent 3 │ │Agent 4 │ │  ...   │     │
│  │        │ │        │ │        │ │        │ │        │     │
│  │/users  │ │/posts  │ │/auth   │ │/search │ │        │     │
│  │endpoint│ │endpoint│ │endpoint│ │endpoint│ │        │     │
│  └───┬────┘ └───┬────┘ └───┬────┘ └───┬────┘ └───┬────┘     │
│      │          │          │          │          │            │
│      └──────────┴──────────┴──────────┴──────────┘            │
│                            │                                   │
│                            ▼                                   │
│                   ┌──────────────┐                             │
│                   │   MERGE +    │                             │
│                   │   VERIFY     │                             │
│                   │              │                             │
│                   │   Check for  │                             │
│                   │   conflicts  │                             │
│                   │   Run tests  │                             │
│                   └──────────────┘                             │
│                                                                │
│  When to use:                                                  │
│  • Many similar, independent tasks                             │
│  • Speed through parallelism                                   │
│  • Tasks share structure but differ in content                 │
└────────────────────────────────────────────────────────────────┘
```

```typescript
// Swarm pattern implementation
async function swarmPattern(tasks: SubTask[]) {
  // All agents work in parallel
  const results = await Promise.all(
    tasks.map(task => 
      agentWorker({
        task: task.description,
        files: task.files,
        constraints: `Only modify files in: ${task.files.join(", ")}. 
                      Do not modify shared files.`
      })
    )
  );

  // Merge phase: combine and resolve conflicts
  const conflicts = detectConflicts(results);
  if (conflicts.length > 0) {
    await resolveConflicts(conflicts);
  }

  // Verification: ensure everything works together
  const testResult = await shell("npm test");
  return { results, testsPassed: testResult.exitCode === 0 };
}
```

## Pattern Comparison

| Dimension | Coordinator | Pipeline | Debate | Swarm |
|-----------|------------|----------|--------|-------|
| **Communication** | Hub-and-spoke | Sequential | Broadcast | Minimal |
| **Parallelism** | High | Low | High | Very high |
| **Consistency** | Coordinator ensures | Each stage validates | Judge synthesizes | Merge step needed |
| **Error propagation** | Coordinator handles | Cascades through stages | Isolated | Isolated |
| **Best for** | Complex features | Review workflows | Architecture decisions | Batch operations |
| **Token cost** | Medium | Low-Medium | High (multiple full proposals) | Medium |

## Coding Applications

| Coding Task | Best Pattern | Why |
|-------------|-------------|-----|
| Implement large feature | **Coordinator** | Needs analysis before delegation, spans domains |
| Code review workflow | **Pipeline** | Natural stages: write → review → fix → test |
| Architecture decision | **Debate** | Multiple valid approaches, want strongest argument |
| Multi-file migration | **Swarm** | Same task repeated across independent files |
| Bug triage + fix | **Pipeline** | Diagnose → locate → fix → verify is sequential |
| Test generation | **Swarm** | Each test file is independent |

## Tradeoffs

```
┌────────────────────────────────────────────────────────────────┐
│              MULTI-AGENT TRADEOFFS                               │
│                                                                │
│  COMMUNICATION OVERHEAD                                        │
│  More agents = more data passing between them                  │
│  Each handoff loses some context (agents don't share memory)   │
│                                                                │
│  CONSISTENCY                                                   │
│  Multiple agents may make inconsistent decisions               │
│  (different naming conventions, different error handling)       │
│  Solution: shared specs, interfaces, conventions document      │
│                                                                │
│  ERROR PROPAGATION                                             │
│  In pipelines: one bad stage corrupts everything downstream    │
│  In swarms: one bad agent may conflict with others             │
│  Solution: verification gates between stages/after merge       │
│                                                                │
│  COST                                                          │
│  N agents × M tokens each = significantly more than 1 agent   │
│  Debate is the most expensive (full proposals from each agent) │
│  Pipeline amortizes cost across focused stages                 │
│                                                                │
│  RULE OF THUMB:                                                │
│  Use multi-agent only when the task benefits clearly from      │
│  decomposition, specialization, or parallel processing.        │
│  A single capable agent is often enough.                       │
└────────────────────────────────────────────────────────────────┘
```

## Next Steps

- [Initializer-Worker Pattern](./initializer-worker.md) — a powerful coordinator variant for code generation
- [Agent Communication](./agent-communication.md) — how agents share context and coordinate
- [Parallel Agent Execution](./parallel-execution.md) — running agents concurrently
- [Orchestrator-Workers](../02-workflow-patterns/orchestrator-workers.md) — the workflow pattern that inspired coordinators
