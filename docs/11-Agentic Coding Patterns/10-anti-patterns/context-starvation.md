# Anti-Pattern: Context Starvation

> When agents run out of context window and quality silently degrades — forgetting decisions, repeating work, and contradicting themselves.

---

## How Context Starvation Happens

Every agentic interaction accumulates context: system prompts, tool calls, file contents, reasoning traces, previous outputs. As the context window fills, the model's ability to attend to all information degrades.

```
Context Window Over Time:

Start    ████░░░░░░░░░░░░░░░░  20% — Clear, focused
         ↓
Mid      ████████████░░░░░░░░  60% — Still functional
         ↓
Late     ██████████████████░░  90% — Quality degrading
         ↓
Full     ████████████████████  100% — Lost recall, contradictions
```

## The U-Shaped Recall Problem

LLMs don't forget uniformly. Research shows a **U-shaped attention curve** — models recall the beginning and end of context well, but lose information in the middle.

```
Recall Quality Across Context:

High ┌─────────────────────────────────────────┐
     │ ██                                  ██  │
     │ ████                              ████  │
     │ ██████                          ██████  │
     │ ████████                      ████████  │
     │ ██████████                  ██████████  │
Low  │ ████████████████████████████████████████│
     └─────────────────────────────────────────┘
      Beginning       Middle              End
      (System      (Earlier tool      (Recent
       prompt)      calls lost)        context)
```

This means:
- **System instructions** (beginning) are generally retained
- **Recent actions** (end) are well-attended
- **Middle context** — earlier decisions, file contents from 30 steps ago, initial requirements — gets lost

## Signs of Context Starvation

| Symptom | What's Happening | Example |
|---------|-----------------|---------|
| **Forgetting earlier decisions** | Middle context lost | Agent re-implements a function it already wrote |
| **Repeating work** | Can't recall previous attempts | Reads the same file again, runs same diagnostic |
| **Contradicting itself** | Conflicting context attended | Says "use approach A" after previously choosing B |
| **Ignoring instructions** | System prompt diluted | Stops following coding conventions mid-session |
| **Declining code quality** | Less context for reasoning | Later changes are sloppier than earlier ones |
| **Hallucinating file contents** | Can't accurately recall files | References functions that don't exist in the file |

## Why Agentic Workflows Are Especially Vulnerable

Standard chat conversations grow linearly. Agentic workflows grow **exponentially** because each step adds:

```
Standard Chat:
  User message    ~100 tokens
  Assistant reply ~500 tokens
  ─────────────────────────
  Per turn:       ~600 tokens

Agentic Workflow:
  Tool call        ~200 tokens
  Tool result      ~2,000 tokens (file contents, command output)
  Reasoning        ~500 tokens
  ─────────────────────────
  Per step:        ~2,700 tokens
  
  10 steps = ~27,000 tokens consumed
  30 steps = ~81,000 tokens consumed (approaching limits)
```

## Prevention Strategies

### Strategy 1: Auto-Compaction

Modern agent tools compact context automatically when it gets too full.

```
Copilot CLI: Auto-compacts at ~95% context usage
  - Summarizes conversation history
  - Preserves system prompt and recent context
  - Drops middle context with summary

Claude Code: Manual /compact command
  - User-triggered compaction
  - Summarizes and restarts with compressed context
```

**Best practice:** Don't wait for auto-compaction. Compact proactively after completing a logical unit of work.

### Strategy 2: Smaller, Focused Sessions

```
❌ BAD: One massive session
┌──────────────────────────────────────────┐
│ Feature A + Feature B + Bug Fix + Refactor│
│ (Context exhausted by Feature B)          │
└──────────────────────────────────────────┘

✅ GOOD: Multiple focused sessions
┌──────────┐ ┌──────────┐ ┌────────┐ ┌──────────┐
│ Feature A│ │ Feature B│ │Bug Fix │ │ Refactor │
│ (Fresh)  │ │ (Fresh)  │ │(Fresh) │ │ (Fresh)  │
└──────────┘ └──────────┘ └────────┘ └──────────┘
```

### Strategy 3: Use Files Instead of Context for State

```markdown
<!-- ❌ BAD: Relying on context to remember state -->
"Remember that we decided to use PostgreSQL, the user table has columns id, 
email, name, and we're using JWT with RS256..."

<!-- ✅ GOOD: Write decisions to a file the agent can re-read -->
## decisions.md
- Database: PostgreSQL
- User table: id (uuid), email (text), name (text)
- Auth: JWT with RS256
```

```typescript
// ✅ GOOD: Agent writes progress to a tracking file
await writeFile("progress.md", `
## Current Status
- [x] Database schema created
- [x] User model implemented  
- [ ] Auth middleware (IN PROGRESS)
- [ ] API routes
`);

// Later, agent reads it back instead of relying on context
const progress = await readFile("progress.md");
```

### Strategy 4: Multi-Window / Multi-Agent Architecture

```
┌─────────────────┐     ┌─────────────────┐
│  Orchestrator    │     │  Worker Agent    │
│  (Tracks plan)   │────▶│  (Fresh context) │
│                  │     │  (Single task)   │
│  Minimal context │     └─────────────────┘
│  - Plan only     │     ┌─────────────────┐
│  - Status only   │────▶│  Worker Agent    │
│                  │     │  (Fresh context) │
└─────────────────┘     └─────────────────┘

Each worker starts with full context budget for its specific task
```

### Strategy 5: Minimal Viable Context

Only load what the agent needs for the current step:

```typescript
// ❌ BAD: Loading entire codebase into context
const allFiles = await glob("src/**/*.ts");
for (const file of allFiles) {
  await readFile(file);  // Each file consumes context
}

// ✅ GOOD: Load only relevant files
const relevantFiles = await grep("function handleAuth", "src/**/*.ts");
for (const file of relevantFiles) {
  await readFile(file);  // Only files that matter
}
```

## Context Management Strategies

| Strategy | When to Use | Effectiveness |
|----------|-------------|---------------|
| **Auto-compaction** | Always on — safety net | High for preventing crashes, medium for quality |
| **Manual /compact** | After completing a logical unit | High — preserves key decisions |
| **Smaller sessions** | Default approach for all work | Very high — fresh context each time |
| **File-based state** | Long-running or multi-session work | Very high — persistent, re-readable |
| **Multi-window** | Complex features requiring many files | High — each agent gets full budget |
| **Progressive loading** | Large codebases | High — only loads what's needed |
| **Explicit summaries** | Before context gets full | Medium — summaries lose detail |

## Context Budget Planning

Before starting a complex task, estimate your context budget:

```
Available context:     200,000 tokens
System prompt:         -5,000 tokens
Instructions/rules:    -3,000 tokens
─────────────────────────────────
Usable budget:         192,000 tokens

Per-step cost (average):
  Tool call + result:  ~2,500 tokens
  Reasoning:           ~500 tokens
  ─────────────────────
  Per step:            ~3,000 tokens

Estimated steps:       192,000 / 3,000 = ~64 steps

Safety margin (80%):   ~50 steps before quality degrades
```

If your task needs more than 50 steps, plan for compaction or session splits.

## Detecting Context Starvation in Practice

```markdown
## Add to Agent Instructions

Monitor your own context usage:
1. If you find yourself re-reading a file you already read, your context may be full
2. If you can't recall a decision you made earlier, check the tracking file
3. After 20+ tool calls, consider whether you need to compact
4. For each new sub-task, state what you know and what you need — 
   if you can't state what you know, context is stale
```
