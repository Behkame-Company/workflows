# Parallel Agent Execution

> Running multiple agents simultaneously for speed — with strategies for partitioning work, preventing conflicts, and merging results.

---

## When to Parallelize

Parallel agent execution trades coordination complexity for speed. It works when tasks are **independent** — different files, different features, different concerns.

```
┌──────────────────────────────────────────────────────────────────┐
│              PARALLEL vs. SEQUENTIAL                              │
│                                                                  │
│  SEQUENTIAL (1 agent):                                          │
│  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐                          │
│  │Task A│→│Task B│→│Task C│→│Task D│  Total: 4 × T             │
│  └──────┘ └──────┘ └──────┘ └──────┘                          │
│                                                                  │
│  PARALLEL (4 agents):                                           │
│  ┌──────┐                                                       │
│  │Task A│──┐                                                    │
│  ├──────┤  │                                                    │
│  │Task B│──┼── all finish ── merge ── verify                    │
│  ├──────┤  │                                                    │
│  │Task C│──┤                          Total: T + merge time     │
│  ├──────┤  │                                                    │
│  │Task D│──┘                                                    │
│  └──────┘                                                       │
│                                                                  │
│  Speedup: ~4x (minus merge/verification overhead)               │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Parallelize When:
- ✅ Tasks operate on **different files** or modules
- ✅ Tasks are **structurally similar** (same pattern, different data)
- ✅ Tasks produce **independent outputs** (no shared state)
- ✅ Multiple **review perspectives** needed (security, performance, correctness)

### Don't Parallelize When:
- ❌ Tasks have **sequential dependencies** (B needs A's output)
- ❌ Tasks modify **shared files** (merge conflicts)
- ❌ Tasks require **shared state** (database connections, global config)
- ❌ Order matters for **correctness** (migrations, state transitions)

## Parallel Execution in Practice

### Copilot CLI: Background Agents

```typescript
// Copilot CLI task tool with mode="background" for parallel agents

// Launch multiple explore agents in parallel — safe, no side effects
const [authAnalysis, dbAnalysis, apiAnalysis] = await Promise.all([
  subAgent({
    type: "explore",
    prompt: "Analyze the authentication module in src/auth/",
    mode: "background"
  }),
  subAgent({
    type: "explore",
    prompt: "Analyze the database layer in src/db/",
    mode: "background"
  }),
  subAgent({
    type: "explore",
    prompt: "Analyze the API routes in src/routes/",
    mode: "background"
  })
]);

// Use all analyses to make an informed plan
const plan = await createPlan(authAnalysis, dbAnalysis, apiAnalysis);
```

### Claude Code: Multiple Terminal Windows

```
┌────────────────────────────────────────────────────────────────┐
│              MULTI-WINDOW PARALLEL EXECUTION                     │
│                                                                │
│  Terminal 1:                                                   │
│  > claude "Implement user auth in src/services/auth.ts.        │
│     Follow interface at src/interfaces/auth.ts"                 │
│                                                                │
│  Terminal 2:                                                   │
│  > claude "Implement user repository in src/repos/user.ts.     │
│     Follow interface at src/interfaces/database.ts"             │
│                                                                │
│  Terminal 3:                                                   │
│  > claude "Implement API routes in src/routes/auth.ts.         │
│     Mock the services for now, wire up later"                   │
│                                                                │
│  All running simultaneously in separate context windows.        │
│  Each agent touches ONLY its assigned files.                   │
└────────────────────────────────────────────────────────────────┘
```

## Strategy 1: Partition by File/Module

The safest approach — each agent owns specific files.

```typescript
interface WorkPartition {
  agentId: string;
  ownedFiles: string[];   // Files this agent can modify
  readOnlyFiles: string[]; // Files this agent can read but not modify
  task: string;
}

async function partitionedExecution(partitions: WorkPartition[]) {
  // Validate: no file is owned by more than one agent
  const allOwned = partitions.flatMap(p => p.ownedFiles);
  const duplicates = allOwned.filter((f, i) => allOwned.indexOf(f) !== i);
  if (duplicates.length > 0) {
    throw new Error(`File conflict: ${duplicates.join(", ")} owned by multiple agents`);
  }

  // Execute all agents in parallel
  const results = await Promise.all(
    partitions.map(partition =>
      subAgent({
        type: "general-purpose",
        prompt: `${partition.task}
                 
                 IMPORTANT CONSTRAINTS:
                 - You may ONLY modify these files: ${partition.ownedFiles.join(", ")}
                 - You may READ these files for context: ${partition.readOnlyFiles.join(", ")}
                 - Do NOT create files outside your owned list
                 - Do NOT modify shared configuration files`
      })
    )
  );

  return results;
}
```

```typescript
// Example: Migrating API endpoints in parallel
const partitions: WorkPartition[] = [
  {
    agentId: "worker-users",
    ownedFiles: ["src/routes/users.ts", "src/routes/users.test.ts"],
    readOnlyFiles: ["src/interfaces/api.ts", "src/middleware/auth.ts"],
    task: "Migrate /users endpoints from REST to GraphQL"
  },
  {
    agentId: "worker-posts",
    ownedFiles: ["src/routes/posts.ts", "src/routes/posts.test.ts"],
    readOnlyFiles: ["src/interfaces/api.ts", "src/middleware/auth.ts"],
    task: "Migrate /posts endpoints from REST to GraphQL"
  },
  {
    agentId: "worker-comments",
    ownedFiles: ["src/routes/comments.ts", "src/routes/comments.test.ts"],
    readOnlyFiles: ["src/interfaces/api.ts", "src/middleware/auth.ts"],
    task: "Migrate /comments endpoints from REST to GraphQL"
  }
];

await partitionedExecution(partitions);
```

## Strategy 2: Interface Contracts

Define shared interfaces before parallel execution to prevent incompatible implementations:

```typescript
// Phase 1: Define contracts (sequential)
async function defineContracts() {
  await writeFile("src/interfaces/service.ts", `
    export interface AuthService {
      login(email: string, password: string): Promise<TokenPair>;
      verify(token: string): Promise<User>;
    }
    
    export interface UserService {
      getById(id: string): Promise<User>;
      create(data: CreateUser): Promise<User>;
    }
  `);
}

// Phase 2: Implement against contracts (parallel)
async function implementInParallel() {
  await Promise.all([
    subAgent({
      type: "general-purpose",
      prompt: `Implement AuthService from src/interfaces/service.ts 
               in src/services/auth.ts. Do NOT modify the interface.`
    }),
    subAgent({
      type: "general-purpose",
      prompt: `Implement UserService from src/interfaces/service.ts 
               in src/services/user.ts. Do NOT modify the interface.`
    })
  ]);
}
```

## Strategy 3: Merge and Resolve

When agents might touch overlapping areas, use git branches and explicit merge resolution:

```
┌────────────────────────────────────────────────────────────────┐
│              BRANCH-BASED PARALLEL EXECUTION                    │
│                                                                │
│  main ─────────────────────────────── merge ── verify          │
│    │                                    ▲                      │
│    ├── branch: agent/auth ─────────────┤                       │
│    │   (Agent A works here)             │                      │
│    │                                    │                      │
│    ├── branch: agent/users ────────────┤                       │
│    │   (Agent B works here)             │                      │
│    │                                    │                      │
│    └── branch: agent/api ──────────────┘                       │
│        (Agent C works here)                                    │
│                                                                │
│  Each agent gets its own branch.                               │
│  Conflicts resolved during merge.                              │
└────────────────────────────────────────────────────────────────┘
```

```typescript
async function branchBasedParallel(tasks: AgentTask[]) {
  const mainBranch = "main";

  // Create a branch for each agent
  for (const task of tasks) {
    await shell(`git checkout -b agent/${task.id} ${mainBranch}`);
    await shell(`git checkout ${mainBranch}`);
  }

  // Execute agents in parallel, each on its own branch
  const results = await Promise.all(
    tasks.map(async task => {
      return await subAgent({
        type: "general-purpose",
        prompt: `You are working on git branch: agent/${task.id}
                 First run: git checkout agent/${task.id}
                 
                 Task: ${task.description}
                 
                 When done, commit all changes to your branch.`
      });
    })
  );

  // Merge all branches
  await shell(`git checkout ${mainBranch}`);
  const mergeConflicts: string[] = [];

  for (const task of tasks) {
    const mergeResult = await shell(`git merge agent/${task.id} --no-edit`);
    if (mergeResult.exitCode !== 0) {
      mergeConflicts.push(task.id);
    }
  }

  // Resolve conflicts if any
  if (mergeConflicts.length > 0) {
    await resolveConflicts(mergeConflicts);
  }

  // Verify merged result
  const tests = await shell("npm test");
  return { results, mergeConflicts, testsPassed: tests.exitCode === 0 };
}
```

## Handling Conflicts

```
┌────────────────────────────────────────────────────────────────┐
│              CONFLICT RESOLUTION STRATEGIES                      │
│                                                                │
│  PREVENTION (best):                                            │
│  • Partition files — no overlap possible                       │
│  • Define interfaces first — implementations can't conflict    │
│  • Use append-only shared files — no overwrite conflicts       │
│                                                                │
│  DETECTION:                                                    │
│  • Git merge — catches text-level conflicts                    │
│  • Type checking — catches interface mismatches                │
│  • Test suite — catches behavioral conflicts                   │
│                                                                │
│  RESOLUTION:                                                   │
│  • Automated — merge tool resolves simple conflicts            │
│  • Agent-assisted — spawn a resolver agent                     │
│  • Sequential fallback — run conflicting tasks sequentially    │
└────────────────────────────────────────────────────────────────┘
```

```typescript
async function resolveConflicts(conflictingAgents: string[]) {
  // Get conflict details
  const conflictFiles = await shell("git diff --name-only --diff-filter=U");
  
  // Spawn a resolver agent
  await subAgent({
    type: "general-purpose",
    prompt: `Git merge conflicts detected in these files:
             ${conflictFiles.stdout}
             
             For each file with conflicts:
             1. Read the file (look for <<<<<<< markers)
             2. Understand both versions
             3. Create a correct merged version
             4. Stage the resolved file with 'git add'
             
             After resolving all conflicts, run 'git commit --no-edit'
             Then run 'npm test' to verify.`
  });
}
```

## Resource Contention

Beyond file conflicts, parallel agents can contend for other resources:

```typescript
// Problem: multiple agents running tests simultaneously
// Both try to use port 3000 for the test server

// Solution: assign different ports per agent
async function parallelWithIsolation(tasks: AgentTask[]) {
  const results = await Promise.all(
    tasks.map((task, i) => 
      subAgent({
        type: "general-purpose",
        prompt: `${task.description}
                 
                 Use PORT=${3000 + i} for any test servers.
                 Use test database: testdb_agent_${i}
                 Prefix temp files with: agent_${i}_`
      })
    )
  );
  return results;
}
```

## Python Implementation

```python
import asyncio
from dataclasses import dataclass

@dataclass
class ParallelTask:
    id: str
    description: str
    owned_files: list[str]
    read_only_files: list[str]

async def parallel_agents(tasks: list[ParallelTask]) -> list[dict]:
    """Run multiple agents in parallel with file partitioning."""

    # Validate no file conflicts
    all_owned = []
    for task in tasks:
        for f in task.owned_files:
            if f in all_owned:
                raise ValueError(f"File {f} owned by multiple agents")
            all_owned.append(f)

    # Run agents concurrently
    async def run_agent(task: ParallelTask) -> dict:
        result = await llm.complete(
            system=f"""You are agent {task.id}.
                      You may ONLY modify: {', '.join(task.owned_files)}
                      You may READ: {', '.join(task.read_only_files)}""",
            tools=[read_file, write_file, edit_file, shell],
            prompt=task.description
        )
        return {"id": task.id, "result": result}

    results = await asyncio.gather(*[run_agent(t) for t in tasks])

    # Verify: run tests
    test_result = await shell("python -m pytest -v")
    if test_result.exit_code != 0:
        await fix_integration_issues(test_result.stderr, tasks)

    return results
```

## Parallel Execution Workflow

```
┌────────────────────────────────────────────────────────────────┐
│              COMPLETE PARALLEL WORKFLOW                           │
│                                                                │
│  1. ANALYZE                                                    │
│     └── Understand the task, identify parallelizable parts     │
│                                                                │
│  2. PARTITION                                                  │
│     └── Assign files/modules to agents                         │
│     └── Define interfaces/contracts                            │
│     └── Verify no conflicts in partition                       │
│                                                                │
│  3. EXECUTE (parallel)                                         │
│     └── Launch agents simultaneously                           │
│     └── Each agent works within its partition                  │
│     └── Agents commit to their branches / files                │
│                                                                │
│  4. MERGE                                                      │
│     └── Combine agent outputs                                  │
│     └── Detect conflicts                                       │
│     └── Resolve conflicts (automated or agent-assisted)        │
│                                                                │
│  5. VERIFY                                                     │
│     └── Run full test suite                                    │
│     └── Type check                                             │
│     └── Lint                                                   │
│     └── If failures: diagnose and fix                          │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

## Guidelines

1. **Partition aggressively** — the cleaner the partition, the fewer conflicts
2. **Interfaces before implementation** — define contracts first, parallelize implementation
3. **Read-only shared files** — agents can read shared files but not modify them
4. **Always verify after merge** — parallel execution introduces subtle integration bugs
5. **Have a sequential fallback** — if parallel fails, fall back to sequential execution
6. **Limit parallelism** — 3-5 agents is the sweet spot; more adds diminishing returns and coordination cost
