# Multi-Agent Context Coordination

> Orchestrating multiple agents while keeping each context window clean and focused.

---

## The Orchestration Challenge

When multiple agents work on related tasks, they need to share information without sharing context:

```
❌ Shared context (polluted):
   All agents see everything → Context rot for all

✅ Coordinated isolation:
   Each agent has clean context → Lead coordinates via summaries
```

---

## Orchestration Patterns

### 1. Fan-Out / Fan-In
```
Lead Agent
  ├── Fan out: Launch N parallel agents
  │     Agent 1: Explore module A → Summary 1
  │     Agent 2: Explore module B → Summary 2
  │     Agent 3: Explore module C → Summary 3
  └── Fan in: Collect summaries → Make decision
```

**Use when**: Independent tasks can run in parallel.

### 2. Pipeline
```
Agent 1 → result → Agent 2 → result → Agent 3 → final result
(research)         (implement)         (test)
```

**Use when**: Each step depends on the previous.

### 3. Supervisor
```
Lead Agent (supervisor):
  - Maintains high-level plan
  - Delegates specific tasks
  - Reviews results
  - Decides next steps

Worker agents:
  - Receive specific, bounded tasks
  - Return structured results
  - Don't know about other workers
```

**Use when**: Complex project requiring ongoing coordination.

---

## Information Passing Between Agents

### Via Lead Agent (Recommended)
```
Agent A → Summary → Lead Agent → Enriched prompt → Agent B
```
Lead filters and curates what Agent B needs from Agent A's results.

### Via Shared Files
```
Agent A → Writes analysis.md → Agent B reads analysis.md
```
Files on disk serve as the shared context. Lead coordinates access.

### Via Structured State
```
Agent A → Updates feature_list.json → Agent B reads it
```
JSON provides structured, schema-enforced data sharing.

---

## Anti-Pattern: Context Contamination

```
❌ Passing Agent A's full raw output to Agent B:
   "Here's everything Agent A found: [5,000 tokens of raw data]..."

✅ Passing curated summary to Agent B:
   "Agent A found that the auth module uses JWT with RS256.
    The bug is in validateToken() at line 45 of jwt.ts.
    You need to fix the key rotation logic."
```

**Rule**: Lead agent curates what sub-agents see. Never pass raw output between sub-agents.

---

## Practical Example

```
Task: "Refactor auth module and update all consumers"

Lead Agent plan:
  1. [Explore agent] Analyze current auth API surface
  2. [Explore agent] Find all consumers of auth module
  3. [Lead decides] New API design based on findings
  4. [Task agent] Implement new auth module
  5. [Task agents, parallel] Update each consumer
  6. [Task agent] Run full test suite
  7. [Lead] Review results, commit
```

Each agent gets a clean context with only the information it needs for its specific task.

---

## Key Takeaways

1. **Lead agent curates** — Never pass raw output between agents
2. **Fan-out for parallel work** — Independent tasks run simultaneously
3. **Pipeline for dependent work** — Each step feeds the next
4. **Files for shared state** — Disk is the shared context
5. **Isolation prevents contamination** — Each agent stays focused

---

## Next Steps

- 🔗 [Minimal Viable Context](../09-best-practices/minimal-viable-context.md) — The guiding principle
- 🔗 [Context Hygiene](../09-best-practices/context-hygiene.md) — Keeping context clean
