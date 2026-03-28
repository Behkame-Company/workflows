# ReAct Pattern

> Reasoning + Acting interleaved — the foundation of modern AI agents.

---

## What Is ReAct?

ReAct (Reasoning + Acting) is a framework where the model alternates between thinking about what to do and actually doing it. Each step has three parts:

```
Thought:     "I need to find which file handles user authentication"
Action:      grep -r "authenticate" src/ 
Observation: Found matches in src/auth/middleware.ts and src/auth/service.ts

Thought:     "The middleware handles route protection, the service handles login logic.
              I should look at the service for the password validation bug."
Action:      read src/auth/service.ts
Observation: [file contents]

Thought:     "Line 42 compares passwords with == instead of using bcrypt.compare()"
Action:      edit src/auth/service.ts line 42
```

---

## ReAct in Development Tools

Modern AI coding tools implement ReAct natively:

| Tool | How ReAct Works |
|------|----------------|
| **GitHub Copilot CLI** | Think → use tools (grep, edit, run tests) → observe → think |
| **Claude Code** | Internal reasoning → bash/file operations → verify |
| **Cursor** | Agentic mode with file editing and terminal |

You don't usually write ReAct prompts manually — the tools implement the loop. But understanding the pattern helps you prompt more effectively.

---

## How to Prompt for ReAct Behavior

### Encouraging Exploration Before Action
```
Before making any changes:
1. Search the codebase to understand the current implementation
2. Read the relevant test files
3. Identify all files that need to change

Then explain your plan before executing it.
```

### Encouraging Verification After Action
```
After making each change:
1. Run the existing tests to check for regressions
2. Verify the change works as intended
3. If tests fail, analyze the failure before making more changes
```

---

## The ReAct Loop for Coding

```
┌─────────────────────────────────────────┐
│  1. OBSERVE: Read relevant files,       │
│     check test results, review errors   │
│              │                          │
│              ▼                          │
│  2. THINK: Analyze the situation,       │
│     form a hypothesis, plan next step   │
│              │                          │
│              ▼                          │
│  3. ACT: Make a change, run a command,  │
│     write code, run tests               │
│              │                          │
│              ▼                          │
│  4. VERIFY: Check the result, compare   │
│     to expected outcome                 │
│              │                          │
│     ┌────────┴────────┐                │
│     │  Success?       │                │
│     │  Yes → Done ✓   │                │
│     │  No → Back to 1 │                │
│     └─────────────────┘                │
└─────────────────────────────────────────┘
```

---

## ReAct vs Pure Reasoning

| Approach | Strengths | Weaknesses |
|----------|-----------|------------|
| Pure reasoning (CoT) | Fast, no tool overhead | Can hallucinate facts |
| Pure acting (no reasoning) | Simple | Random actions, no strategy |
| ReAct (combined) | Grounded in real data | Slower, uses more tokens |

**For coding tasks**: ReAct almost always wins because it grounds the model in actual file contents and test results instead of guessing.

---

## Prompting Tips for ReAct Agents

1. **Give the agent verification tools**: "Run tests after every change"
2. **Encourage incremental progress**: "Make one change at a time and verify"
3. **Set exploration boundaries**: "Only look at files in src/auth/"
4. **Require evidence**: "Don't assume what a file contains — read it first"

---

## Next Steps

- 🔗 [Tool-Augmented Prompting](tool-augmented-prompting.md) — Prompting models to use tools
- 🔗 [Agentic Prompting](agentic-prompting.md) — Autonomous agent patterns
