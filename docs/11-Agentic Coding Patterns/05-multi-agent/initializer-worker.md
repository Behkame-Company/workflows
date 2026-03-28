# Initializer-Worker Pattern

> One agent scaffolds the project structure and defines interfaces, then worker agents implement individual components against those contracts — enabling focused context and parallel implementation.

---

## The Pattern

The initializer-worker pattern splits code generation into two distinct phases:

1. **Initializer**: Creates project structure, defines interfaces, writes specifications
2. **Workers**: Implement individual components against the interfaces the initializer defined

Each worker has **focused context** — it doesn't need to understand the whole project, just its interfaces and its assigned component.

```
┌──────────────────────────────────────────────────────────────────┐
│                 INITIALIZER-WORKER PATTERN                        │
│                                                                  │
│  ┌───────────────────────────────────────────────────┐          │
│  │                  INITIALIZER                       │          │
│  │                                                    │          │
│  │  1. Analyze task requirements                      │          │
│  │  2. Design overall architecture                    │          │
│  │  3. Create directory structure                     │          │
│  │  4. Define interfaces / type contracts             │          │
│  │  5. Write integration points                       │          │
│  │  6. Create specification for each component        │          │
│  │                                                    │          │
│  │  Outputs:                                          │          │
│  │  ├── src/interfaces/           (type contracts)    │          │
│  │  ├── src/index.ts              (wiring/integration)│          │
│  │  ├── specs/component-a.md      (worker specs)      │          │
│  │  └── specs/component-b.md                          │          │
│  └────────────────────┬──────────────────────────────┘          │
│                       │                                          │
│         ┌─────────────┼─────────────┐                           │
│         ▼             ▼             ▼                           │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                     │
│  │ WORKER A │  │ WORKER B │  │ WORKER C │                     │
│  │          │  │          │  │          │                     │
│  │ Reads:   │  │ Reads:   │  │ Reads:   │                     │
│  │ interface│  │ interface│  │ interface│                     │
│  │ + spec A │  │ + spec B │  │ + spec C │                     │
│  │          │  │          │  │          │                     │
│  │ Writes:  │  │ Writes:  │  │ Writes:  │                     │
│  │ auth.ts  │  │ db.ts    │  │ api.ts   │                     │
│  │ auth.    │  │ db.      │  │ api.     │                     │
│  │ test.ts  │  │ test.ts  │  │ test.ts  │                     │
│  └──────────┘  └──────────┘  └──────────┘                     │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

## Why This Works

```
┌────────────────────────────────────────────────────────────────┐
│           WHY INITIALIZER-WORKER IS POWERFUL                    │
│                                                                │
│  1. FOCUSED CONTEXT                                            │
│     Each worker only needs:                                    │
│     • Its interface contract                                   │
│     • Its component spec                                       │
│     • NOT the entire project                                   │
│     → Fits in context window, reduces confusion                │
│                                                                │
│  2. CLEAR CONTRACTS                                            │
│     Interfaces define inputs/outputs explicitly                │
│     → Workers can't accidentally create incompatible APIs      │
│                                                                │
│  3. PARALLEL IMPLEMENTATION                                    │
│     Workers can run simultaneously (different files)           │
│     → 3 workers = ~3x faster than sequential                   │
│                                                                │
│  4. MODULAR VERIFICATION                                       │
│     Each component can be tested against its interface         │
│     → Failures are localized                                   │
│                                                                │
│  5. CLEAN ARCHITECTURE                                         │
│     Separating design from implementation forces good          │
│     architecture thinking upfront                              │
└────────────────────────────────────────────────────────────────┘
```

## Implementation

### The Initializer Agent

```typescript
async function initializerAgent(task: string): Promise<InitializerOutput> {
  const output = await llm.complete({
    system: `You are a senior software architect. Your job is to:
             1. Analyze the task and design the architecture
             2. Create the directory structure
             3. Define TypeScript interfaces for all components
             4. Write a spec for each component that a junior developer 
                could implement independently
             
             Do NOT implement the components — only define the structure 
             and contracts. Workers will handle implementation.
             
             For each interface, include:
             - All type definitions
             - Function signatures with JSDoc
             - Expected behavior descriptions
             - Error cases to handle`,
    tools: [readFile, writeFile, search, listDir, shell],
    prompt: `Task: ${task}
             
             1. Explore the existing codebase
             2. Design the architecture
             3. Create interfaces in src/interfaces/
             4. Create component specs in .specs/
             5. Create the integration entry point`
  });

  // Parse what the initializer created
  return {
    interfaces: await glob("src/interfaces/*.ts"),
    specs: await glob(".specs/*.md"),
    entryPoint: "src/index.ts"
  };
}
```

### Example: Initializer Output

```typescript
// src/interfaces/auth.ts — Created by initializer
export interface AuthService {
  /**
   * Authenticate a user with email and password.
   * @returns JWT token pair on success
   * @throws AuthError with code 'INVALID_CREDENTIALS' if auth fails
   * @throws AuthError with code 'ACCOUNT_LOCKED' after 5 failed attempts
   */
  login(email: string, password: string): Promise<TokenPair>;

  /**
   * Refresh an expired access token using a valid refresh token.
   * @throws AuthError with code 'INVALID_REFRESH_TOKEN' if token is invalid/expired
   */
  refresh(refreshToken: string): Promise<TokenPair>;

  /** Invalidate all tokens for a user (logout everywhere) */
  revokeAll(userId: string): Promise<void>;
}

export interface TokenPair {
  accessToken: string;   // JWT, expires in 15 minutes
  refreshToken: string;  // Opaque token, expires in 7 days
}

export interface AuthError extends Error {
  code: 'INVALID_CREDENTIALS' | 'ACCOUNT_LOCKED' | 'INVALID_REFRESH_TOKEN';
}
```

```typescript
// src/interfaces/database.ts — Created by initializer
export interface UserRepository {
  findByEmail(email: string): Promise<User | null>;
  findById(id: string): Promise<User | null>;
  create(data: CreateUserData): Promise<User>;
  updateFailedAttempts(userId: string, count: number): Promise<void>;
}

export interface User {
  id: string;
  email: string;
  passwordHash: string;
  failedLoginAttempts: number;
  lockedUntil: Date | null;
}
```

### The Worker Agent

```typescript
async function workerAgent(
  componentSpec: string, 
  interfaces: string[],
  constraints: string[]
): Promise<WorkerResult> {
  // Load the spec and relevant interfaces
  const spec = await readFile(componentSpec);
  const interfaceCode = await Promise.all(interfaces.map(f => readFile(f)));

  const result = await llm.complete({
    system: `You are implementing a single component. 
             Follow the interface contract EXACTLY — do not modify interfaces.
             
             Your implementation must:
             - Satisfy all interface requirements
             - Handle all error cases described in the interface
             - Include unit tests
             - Follow existing project conventions
             
             Only create files in your assigned scope.`,
    tools: [readFile, writeFile, editFile, shell],
    prompt: `Component spec:\n${spec}
             
             Interfaces you must implement:\n${interfaceCode.join("\n\n")}
             
             Constraints:\n${constraints.join("\n")}
             
             Implement the component, write tests, and verify tests pass.`
  });

  return {
    files: extractCreatedFiles(result),
    testsPassed: await runComponentTests(componentSpec)
  };
}
```

### Orchestrating the Full Flow

```typescript
async function initializerWorkerFlow(task: string) {
  console.log("Phase 1: Initializer creating architecture...");
  const scaffold = await initializerAgent(task);
  
  console.log(`Phase 2: Spawning ${scaffold.specs.length} workers...`);
  
  // Workers execute in parallel — they touch different files
  const workerResults = await Promise.all(
    scaffold.specs.map(spec => 
      workerAgent(spec, scaffold.interfaces, [
        `Do not modify files in src/interfaces/`,
        `Do not modify ${scaffold.entryPoint}`
      ])
    )
  );

  console.log("Phase 3: Integration verification...");
  
  // Run full test suite to verify components work together
  const integrationResult = await shell("npm test");
  
  if (integrationResult.exitCode !== 0) {
    console.log("Integration issues detected, running fix-up agent...");
    await fixIntegrationIssues(integrationResult.stderr, scaffold);
  }

  return {
    scaffold,
    workers: workerResults,
    integrationPassed: integrationResult.exitCode === 0
  };
}
```

## In Practice: Copilot CLI

Copilot CLI's task tool with sub-agents implements this pattern directly:

```typescript
// Main agent acts as initializer
const plan = await createArchitecture(task);

// Sub-agents act as workers
await Promise.all([
  subAgent({
    type: "general-purpose",
    prompt: `Implement the auth service following the interface at src/interfaces/auth.ts.
             Read the spec at .specs/auth-service.md.
             Write implementation to src/services/auth.ts and tests to src/services/auth.test.ts.
             Do not modify any interface files.`
  }),
  subAgent({
    type: "general-purpose",
    prompt: `Implement the user repository following the interface at src/interfaces/database.ts.
             Read the spec at .specs/user-repository.md.
             Write implementation to src/repositories/user.ts and tests to src/repositories/user.test.ts.
             Do not modify any interface files.`
  }),
  subAgent({
    type: "general-purpose",
    prompt: `Implement the API routes following the spec at .specs/api-routes.md.
             Use the auth service interface at src/interfaces/auth.ts.
             Write routes to src/routes/auth.ts and tests to src/routes/auth.test.ts.
             Do not modify any interface files.`
  })
]);
```

## In Practice: Claude Code Multi-Window

```
┌────────────────────────────────────────────────────────────────┐
│              CLAUDE CODE MULTI-WINDOW WORKFLOW                   │
│                                                                │
│  Terminal 1 (Initializer):                                     │
│  > "Analyze the task. Create interfaces in src/interfaces/     │
│     and component specs in .specs/. Don't implement anything." │
│                                                                │
│  ... initializer finishes ...                                  │
│                                                                │
│  Terminal 2 (Worker A):                                        │
│  > "Read .specs/auth.md and src/interfaces/auth.ts.            │
│     Implement the auth service. Run tests."                    │
│                                                                │
│  Terminal 3 (Worker B):                                        │
│  > "Read .specs/database.md and src/interfaces/database.ts.    │
│     Implement the user repository. Run tests."                 │
│                                                                │
│  Terminal 4 (Worker C):                                        │
│  > "Read .specs/api.md and relevant interfaces.                │
│     Implement the API routes. Run tests."                      │
│                                                                │
│  ... all workers finish in parallel ...                        │
│                                                                │
│  Terminal 1 (Integration):                                     │
│  > "All components are implemented. Run the full test suite    │
│     and fix any integration issues."                           │
└────────────────────────────────────────────────────────────────┘
```

## Design Guidelines

1. **Interfaces should be complete** — workers shouldn't need to guess. Include types, error cases, behavior descriptions.
2. **Specs should be standalone** — each worker spec should contain everything needed to implement the component without reading other specs.
3. **Workers should have file boundaries** — explicitly define which files each worker owns to prevent conflicts.
4. **Include integration tests** — unit tests per component aren't enough; test the components together.
5. **Keep the initializer lean** — it designs, it doesn't implement. If it starts coding, it's doing the workers' job.

## When to Use Initializer-Worker

| Situation | Use Initializer-Worker? |
|-----------|------------------------|
| New feature with 3+ components | ✅ Yes — parallel workers save time |
| Single-file change | ❌ No — overhead not worth it |
| Refactor with many similar files | ✅ Yes — same interface, many workers |
| Exploratory debugging | ❌ No — need iterative exploration |
| API + frontend + tests | ✅ Yes — natural separation by domain |

## Next Steps

- [Multi-Agent Patterns](./multi-agent-patterns.md) — the four primary multi-agent architectures
- [Agent Communication](./agent-communication.md) — how initializer and workers share context
- [Parallel Agent Execution](./parallel-execution.md) — running workers concurrently
- [Orchestrator-Workers](../02-workflow-patterns/orchestrator-workers.md) — the related workflow pattern
