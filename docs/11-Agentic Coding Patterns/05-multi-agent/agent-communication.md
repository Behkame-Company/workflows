# Agent Communication

> How agents share context and coordinate when they don't share memory — using files, structured data, git, and explicit message passing.

---

## The Core Challenge

Each agent operates in its own **isolated context window**. Unlike human teams who share knowledge through conversation and shared documents, agents have no built-in shared memory. Context must be **explicitly passed** through external channels.

```
┌──────────────────────────────────────────────────────────────────┐
│               THE AGENT COMMUNICATION CHALLENGE                   │
│                                                                  │
│  ┌─────────────────┐          ┌─────────────────┐              │
│  │    AGENT A       │          │    AGENT B       │              │
│  │                  │    ?     │                  │              │
│  │  Context Window  │◄────────►│  Context Window  │              │
│  │  ┌────────────┐ │          │ ┌────────────┐  │              │
│  │  │ Its own    │ │          │ │ Its own    │  │              │
│  │  │ history    │ │  No      │ │ history    │  │              │
│  │  │ Its own    │ │  shared  │ │ Its own    │  │              │
│  │  │ tool calls │ │  memory! │ │ tool calls │  │              │
│  │  │ Its own    │ │          │ │ Its own    │  │              │
│  │  │ reasoning  │ │          │ │ reasoning  │  │              │
│  │  └────────────┘ │          │ └────────────┘  │              │
│  └─────────────────┘          └─────────────────┘              │
│                                                                  │
│  Solution: External communication channels                       │
│                                                                  │
│  ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐           │
│  │Files │  │ JSON │  │ Git  │  │Memory│  │ Task │           │
│  │      │  │ /YAML│  │      │  │ Files│  │Result│           │
│  └──────┘  └──────┘  └──────┘  └──────┘  └──────┘           │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

## Communication Channels

### Channel 1: File System

The simplest and most reliable channel. Agents write files; other agents read them.

```
┌────────────────────────────────────────────────────────────────┐
│              FILE-BASED COMMUNICATION                            │
│                                                                │
│  Initializer Agent                Worker Agent A               │
│  ─────────────────                ──────────────               │
│                                                                │
│  Writes:                          Reads:                       │
│  ├── specs/auth.md      ────────► specs/auth.md               │
│  ├── src/interfaces/    ────────► src/interfaces/             │
│  └── architecture.md    ────────► architecture.md             │
│                                                                │
│                                   Writes:                      │
│                                   ├── src/services/auth.ts     │
│                                   └── src/services/auth.test.ts│
│                                                                │
│  Advantages:                                                   │
│  • Persistent — survives agent crashes                         │
│  • Debuggable — humans can read the files too                  │
│  • Natural — agents already know how to read/write files       │
│                                                                │
│  Disadvantages:                                                │
│  • No notification — reader must know when to read             │
│  • No structure enforcement — writer might use wrong format    │
└────────────────────────────────────────────────────────────────┘
```

```typescript
// File-based communication pattern
async function fileBasedComm() {
  // Initializer writes spec
  await writeFile("specs/auth.md", `
    # Auth Service Spec
    
    ## Interface
    Implement AuthService from src/interfaces/auth.ts
    
    ## Requirements
    - Password hashing with bcrypt (cost factor 12)
    - JWT access tokens expire in 15 minutes
    - Refresh tokens stored in database
    - Lock account after 5 failed attempts for 30 minutes
    
    ## Files to Create
    - src/services/auth.ts (implementation)
    - src/services/auth.test.ts (tests)
  `);

  // Worker reads spec and implements
  const spec = await readFile("specs/auth.md");
  await implementComponent(spec);
}
```

### Channel 2: Structured Data (JSON/YAML)

For machine-readable communication between agents, use structured formats:

```typescript
// tests-tracker.json — shared between agents
interface TestTracker {
  components: {
    [name: string]: {
      status: "pending" | "implementing" | "testing" | "done" | "failed";
      assignedTo: string;
      files: string[];
      testResults?: {
        passed: number;
        failed: number;
        errors: string[];
      };
    };
  };
  lastUpdated: string;
}

// Worker A updates its status
async function updateTrackerStatus(component: string, status: string, results?: any) {
  const tracker: TestTracker = JSON.parse(await readFile("tracker.json"));
  
  tracker.components[component].status = status;
  if (results) {
    tracker.components[component].testResults = results;
  }
  tracker.lastUpdated = new Date().toISOString();
  
  await writeFile("tracker.json", JSON.stringify(tracker, null, 2));
}

// Coordinator checks overall progress
async function checkProgress(): Promise<ProgressReport> {
  const tracker: TestTracker = JSON.parse(await readFile("tracker.json"));
  
  const total = Object.keys(tracker.components).length;
  const done = Object.values(tracker.components).filter(c => c.status === "done").length;
  const failed = Object.values(tracker.components).filter(c => c.status === "failed");
  
  return { total, done, failed: failed.length, details: tracker };
}
```

```json
// tracker.json — the communication artifact
{
  "components": {
    "auth-service": {
      "status": "done",
      "assignedTo": "worker-1",
      "files": ["src/services/auth.ts", "src/services/auth.test.ts"],
      "testResults": {
        "passed": 12,
        "failed": 0,
        "errors": []
      }
    },
    "user-repository": {
      "status": "implementing",
      "assignedTo": "worker-2",
      "files": ["src/repositories/user.ts"]
    },
    "api-routes": {
      "status": "pending",
      "assignedTo": "worker-3",
      "files": []
    }
  },
  "lastUpdated": "2025-01-15T10:30:00Z"
}
```

### Channel 3: Git as Communication

Git commits serve as **progress markers** that other agents can read:

```
┌────────────────────────────────────────────────────────────────┐
│              GIT-BASED COMMUNICATION                             │
│                                                                │
│  Worker A commits:                                             │
│  ┌──────────────────────────────────────┐                     │
│  │ commit abc123                         │                     │
│  │ "feat(auth): implement login flow     │                     │
│  │                                       │                     │
│  │  Implements AuthService.login()       │                     │
│  │  - bcrypt password hashing            │                     │
│  │  - JWT token generation               │                     │
│  │  - Account lockout after 5 fails      │                     │
│  │                                       │                     │
│  │  Tests: 12 passed, 0 failed"          │                     │
│  └──────────────────────────────────────┘                     │
│                                                                │
│  Integration agent reads:                                      │
│  $ git log --oneline --since="1 hour ago"                      │
│  abc123 feat(auth): implement login flow                       │
│  def456 feat(db): implement user repository                    │
│  → "Both components done, time for integration testing"        │
│                                                                │
│  Advantages:                                                   │
│  • Natural progress tracking                                   │
│  • Built-in rollback (git revert)                              │
│  • Commit messages = structured communication                  │
└────────────────────────────────────────────────────────────────┘
```

```typescript
// Git-based progress communication
async function commitProgress(component: string, summary: string) {
  await shell("git add -A");
  await shell(`git commit -m "feat(${component}): ${summary}

Implements ${component} component.
Status: complete
Tests: all passing"`);
}

// Reader: check what other agents have done
async function checkAgentProgress(): Promise<string[]> {
  const log = await shell('git log --oneline --since="2 hours ago"');
  return log.stdout.trim().split("\n");
}
```

### Channel 4: Memory Files

Persistent files that accumulate context across agent sessions (CLAUDE.md, .copilot-instructions.md):

```markdown
<!-- .agent-memory/decisions.md — accumulated by agents over time -->
# Architecture Decisions

## 2025-01-15: Auth Strategy
Decision: JWT with refresh tokens
Rationale: Stateless auth for horizontal scaling
Agent: initializer

## 2025-01-15: Database ORM
Decision: Prisma over raw SQL
Rationale: Type safety, migration management, existing project convention
Agent: initializer

## 2025-01-15: Error Handling Pattern
Decision: Custom error classes extending base AppError
Rationale: Consistent error responses across all endpoints
Agent: worker-api-routes
```

```typescript
// Agents append decisions to shared memory
async function recordDecision(decision: string, rationale: string, agent: string) {
  const entry = `\n## ${new Date().toISOString().split("T")[0]}: ${decision}
Decision: ${decision}
Rationale: ${rationale}
Agent: ${agent}\n`;

  await appendToFile(".agent-memory/decisions.md", entry);
}

// Other agents read accumulated decisions
async function getDecisions(): Promise<string> {
  return await readFile(".agent-memory/decisions.md");
}
```

### Channel 5: Task Results

Direct output passing — one agent's output becomes the next agent's input:

```typescript
// Task tool: agent output flows to caller
const analysisResult = await subAgent({
  type: "explore",
  prompt: "Analyze the codebase structure and identify all API endpoints"
});

// The result is immediately available as context for the next agent
const implementationResult = await subAgent({
  type: "general-purpose",
  prompt: `Based on this analysis:
           ${analysisResult}
           
           Implement rate limiting for all identified endpoints.`
});
```

## Communication Patterns

### Pattern 1: Spec File Handoff

```
┌────────────────────────────────────────────────────────────────┐
│              SPEC FILE HANDOFF                                   │
│                                                                │
│  Agent A writes:           Agent B reads:                      │
│  ┌──────────────┐         ┌──────────────┐                    │
│  │ spec.md      │────────►│ spec.md      │                    │
│  │              │         │              │                    │
│  │ Requirements │         │ Requirements │ → Implements       │
│  │ Interfaces   │         │ Interfaces   │ → Tests            │
│  │ Constraints  │         │ Constraints  │ → Verifies         │
│  └──────────────┘         └──────────────┘                    │
│                                                                │
│  Best for: Initializer-worker, pipeline stages                 │
└────────────────────────────────────────────────────────────────┘
```

### Pattern 2: Shared Tracker File

```
┌────────────────────────────────────────────────────────────────┐
│              SHARED TRACKER                                      │
│                                                                │
│  ┌──────────────────┐                                         │
│  │   tracker.json    │◄──── All agents read/write             │
│  │                   │                                         │
│  │  {                │      Agent A: updates auth status       │
│  │    "auth": "done" │      Agent B: checks if auth is done   │
│  │    "db": "wip"    │      Agent C: waits for db to finish   │
│  │    "api": "wait"  │      Coordinator: monitors overall     │
│  │  }                │                                         │
│  └──────────────────┘                                         │
│                                                                │
│  Best for: Swarm coordination, progress monitoring             │
│  ⚠ Caution: concurrent writes can cause conflicts              │
└────────────────────────────────────────────────────────────────┘
```

### Pattern 3: Message Queue (via Files)

```typescript
// Simple file-based message queue
const QUEUE_DIR = ".agent-messages";

async function sendMessage(from: string, to: string, message: any) {
  const id = `${Date.now()}-${Math.random().toString(36).slice(2)}`;
  const msg = { id, from, to, timestamp: new Date().toISOString(), ...message };
  
  await writeFile(
    `${QUEUE_DIR}/${to}/${id}.json`,
    JSON.stringify(msg, null, 2)
  );
}

async function readMessages(agentId: string): Promise<any[]> {
  const dir = `${QUEUE_DIR}/${agentId}`;
  const files = await glob(`${dir}/*.json`);
  
  const messages = await Promise.all(
    files.map(async f => {
      const content = JSON.parse(await readFile(f));
      await deleteFile(f); // Consume the message
      return content;
    })
  );

  return messages.sort((a, b) => a.timestamp.localeCompare(b.timestamp));
}
```

## Communication Flow Diagrams

### Pipeline Communication

```
┌────────────────────────────────────────────────────────────────┐
│              PIPELINE COMMUNICATION FLOW                         │
│                                                                │
│  Planner ──── spec.md ────► Coder ──── code ────► Reviewer    │
│                                                       │        │
│                                                  review.md     │
│                                                       │        │
│  Coder ◄──── review.md ──── Reviewer                 │        │
│    │                                                  │        │
│    └──── fixed code ────► Reviewer (re-review) ──────►│        │
│                                                       │        │
│                                    ┌──── code ────► Tester     │
│                                    │                  │        │
│                                    │            test-results   │
│                                    │                  │        │
│                                    └◄── results ──────┘        │
└────────────────────────────────────────────────────────────────┘
```

### Swarm Communication

```
┌────────────────────────────────────────────────────────────────┐
│              SWARM COMMUNICATION FLOW                             │
│                                                                │
│           ┌──────────────────┐                                 │
│           │   tracker.json    │                                 │
│           └────────┬─────────┘                                 │
│         ┌──────────┼──────────┐                                │
│         │          │          │                                 │
│         ▼          ▼          ▼                                 │
│    ┌─────────┐┌─────────┐┌─────────┐                          │
│    │Worker 1 ││Worker 2 ││Worker 3 │                          │
│    │         ││         ││         │                          │
│    │ Read    ││ Read    ││ Read    │                          │
│    │ tracker ││ tracker ││ tracker │                          │
│    │         ││         ││         │                          │
│    │ Do work ││ Do work ││ Do work │                          │
│    │         ││         ││         │                          │
│    │ Update  ││ Update  ││ Update  │                          │
│    │ tracker ││ tracker ││ tracker │                          │
│    └─────────┘└─────────┘└─────────┘                          │
│         │          │          │                                 │
│         └──────────┼──────────┘                                │
│                    ▼                                            │
│           ┌──────────────────┐                                 │
│           │   Coordinator    │                                 │
│           │   reads tracker  │                                 │
│           │   when all done  │                                 │
│           └──────────────────┘                                 │
└────────────────────────────────────────────────────────────────┘
```

## Best Practices

1. **Be explicit** — don't assume agents share context. Pass everything they need through the channel.
2. **Use structured formats** — JSON/YAML for machine consumption, Markdown for mixed human/agent use.
3. **Include metadata** — timestamps, agent IDs, and status help with debugging and coordination.
4. **Design for idempotency** — agents may read the same file twice; ensure this is safe.
5. **Keep messages small** — large messages eat into the receiving agent's context window.
6. **Test the communication** — verify that Agent B can actually use what Agent A produces.

## Next Steps

- [Multi-Agent Patterns](./multi-agent-patterns.md) — the architectures these channels support
- [Initializer-Worker Pattern](./initializer-worker.md) — the pattern that relies most on file-based communication
- [Parallel Agent Execution](./parallel-execution.md) — coordinating concurrent agents
- [Tool Design Principles](../04-tool-use/tool-design-principles.md) — designing the tools agents communicate through
