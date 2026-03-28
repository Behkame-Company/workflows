# Transparency and Observability

> Anthropic's #2 principle: explicitly show the agent's planning steps — agents that explain their reasoning are easier to debug, steer, and trust.

---

## Why Transparency Matters

Opaque agents are dangerous agents. When you can't see what an agent is thinking or doing, you can't:
- **Debug** when it goes wrong
- **Steer** when it's heading in the wrong direction
- **Trust** that it's doing what you asked
- **Learn** from its approach to improve future prompts

```
┌──────────────────────────────────────────────────────────────┐
│           Opaque vs Transparent Agent                         │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│   Opaque Agent                 Transparent Agent             │
│   ────────────                 ─────────────────             │
│   "Task complete."             "Here's my plan:              │
│                                 1. Read the user service     │
│   Did it work?                  2. Add validation            │
│   What did it do?               3. Write tests               │
│   Why that approach?            4. Run test suite             │
│   Any side effects?                                          │
│                                 Step 1: Reading              │
│   🤷 No idea.                  src/services/user.ts...       │
│                                 Found: no input validation   │
│                                 on createUser method.        │
│                                                              │
│                                 Step 2: Adding Zod schema..."│
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

## The Four Pillars of Agent Observability

### 1. Show the Plan Before Execution

```
┌──────────────────────────────────────────────────┐
│        Plan → Approve → Execute                   │
├──────────────────────────────────────────────────┤
│                                                   │
│  Agent creates plan                               │
│       │                                           │
│       ▼                                           │
│  Human reviews plan                               │
│       │                                           │
│       ├── Approve ──▶ Agent executes              │
│       │                                           │
│       ├── Modify  ──▶ Agent revises plan          │
│       │                                           │
│       └── Reject  ──▶ Agent tries new approach    │
│                                                   │
└──────────────────────────────────────────────────┘
```

**Copilot CLI implements this:** Use Shift+Tab for plan mode. The agent outlines its approach before making any changes.

**Claude Code implements this:** Shows thinking and reasoning before tool invocations. You can approve or redirect.

### 2. Log Tool Calls and Results

Every action the agent takes should be visible:

```
┌──────────────────────────────────────────────────────────────┐
│               Tool Call Logging                               │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  [14:23:01] Tool: ReadFile                                   │
│            Path: src/services/user.ts                        │
│            Result: 247 lines read                            │
│                                                              │
│  [14:23:03] Tool: Search                                     │
│            Pattern: "createUser"                             │
│            Result: 3 matches in 2 files                      │
│                                                              │
│  [14:23:05] Tool: EditFile                                   │
│            Path: src/services/user.ts                        │
│            Change: Added Zod validation schema               │
│                                                              │
│  [14:23:08] Tool: Shell                                      │
│            Command: npm test -- --testPathPattern=user       │
│            Result: 12 tests passed, 0 failed                 │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

Both Copilot CLI and Claude Code show tool invocations in real time. This is a feature — don't hide it.

### 3. Display Progress During Long Operations

For multi-step tasks, show where the agent is in its workflow:

```
┌──────────────────────────────────────────────────────────────┐
│              Progress Reporting                               │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│   Feature: Add webhook support                               │
│                                                              │
│   [✓] Step 1/6: Analyze existing event system                │
│   [✓] Step 2/6: Create webhook types                         │
│   [▶] Step 3/6: Implement WebhookService                     │
│   [ ] Step 4/6: Create API endpoints                         │
│   [ ] Step 5/6: Write tests                                  │
│   [ ] Step 6/6: Run full test suite                          │
│                                                              │
│   Current: Writing send() method with retry logic            │
│   Files modified: 2  │  Tests: not yet run                   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### 4. Make Decision Points Visible

When the agent makes a choice, it should explain why:

```markdown
## Decision: Using Bull for job queue

Considered:
1. Bull (Redis-based) — already in package.json, team is familiar
2. BullMQ — newer but would add a dependency
3. Custom queue — simpler but no retry/backoff built in

Chose: Bull — project already uses it for email jobs,
consistent with existing patterns.
```

## Building Observable Agents

### Structured Logging

When building custom agent workflows, add structured logging:

```typescript
interface AgentLog {
  timestamp: string;
  phase: "plan" | "execute" | "verify";
  action: string;
  tool?: string;
  input?: string;
  output?: string;
  decision?: {
    options: string[];
    chosen: string;
    reason: string;
  };
}

// Example log entries
const logs: AgentLog[] = [
  {
    timestamp: "2024-01-15T14:23:01Z",
    phase: "plan",
    action: "Analyzing codebase for validation patterns",
    tool: "grep",
    input: "Zod|yup|joi",
    output: "Found 12 Zod schemas in src/validators/"
  },
  {
    timestamp: "2024-01-15T14:23:03Z",
    phase: "plan",
    action: "Choosing validation library",
    decision: {
      options: ["Zod", "Yup", "Joi"],
      chosen: "Zod",
      reason: "12 existing schemas use Zod - maintaining consistency"
    }
  }
];
```

### Progress Reporting

```typescript
interface ProgressReport {
  taskId: string;
  totalSteps: number;
  currentStep: number;
  stepName: string;
  status: "in-progress" | "complete" | "failed";
  filesModified: string[];
  testsRun: number;
  testsPassed: number;
}
```

### Decision Audit Trail

```typescript
interface Decision {
  id: string;
  context: string;           // What prompted this decision
  options: {
    name: string;
    pros: string[];
    cons: string[];
  }[];
  chosen: string;            // Which option was selected
  reason: string;            // Why this option
  reversible: boolean;       // Can this be undone?
  timestamp: string;
}
```

## How Current Tools Implement Transparency

### Copilot CLI

- Shows each tool call before execution
- Asks permission before file modifications and shell commands
- Displays reasoning alongside actions
- Plan mode (Shift+Tab) for preview without execution

### Claude Code

- Shows thinking process in extended thinking
- Displays tool invocations with parameters
- Permission prompts for sensitive operations
- Approval/rejection with inline feedback

## Observability Checklist

Use this checklist when building or evaluating agent systems:

```
┌──────────────────────────────────────────────────────────────┐
│           Observability Checklist                             │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Planning                                                    │
│  [ ] Agent shows plan before executing                       │
│  [ ] Plan includes specific files and actions                │
│  [ ] Human can approve, modify, or reject plan               │
│                                                              │
│  Execution                                                   │
│  [ ] Each tool call is logged with inputs and outputs        │
│  [ ] Progress is visible during multi-step operations        │
│  [ ] Errors are surfaced immediately, not hidden             │
│                                                              │
│  Decision Making                                             │
│  [ ] Agent explains choices between alternatives             │
│  [ ] Reasoning is logged, not just actions                   │
│  [ ] Decision points are marked for human review             │
│                                                              │
│  Post-Execution                                              │
│  [ ] Summary of all changes made                             │
│  [ ] List of files created/modified/deleted                  │
│  [ ] Test results included                                   │
│  [ ] Next steps or open questions documented                 │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

## The Trust Gradient

Transparency enables a trust gradient — start with high oversight and reduce as confidence grows:

```
High Oversight ◀──────────────────────────▶ Low Oversight

  Approve every        Approve tool        Allow all tools,
  tool call            categories          review results
  ┌────────┐          ┌────────┐          ┌────────┐
  │ Day 1  │   ───▶   │ Week 2 │   ───▶   │Month 2 │
  │        │          │        │          │        │
  │ Watch  │          │ Spot   │          │ Review │
  │ every  │          │ check  │          │ output │
  │ action │          │        │          │ only   │
  └────────┘          └────────┘          └────────┘
```

You can only reduce oversight when you can **observe** that the agent is consistently doing the right thing. Without transparency, you're stuck at high oversight forever — or taking blind risks.

## Next Steps

- [Error Recovery](./error-recovery.md) — transparent systems recover better
- [Testing Agentic Systems](./testing-agents.md) — verify agent behavior systematically
- [Blind Delegation](../10-anti-patterns/blind-delegation.md) — what happens without transparency
- [Start Simple](./start-simple.md) — simpler systems are inherently more transparent
