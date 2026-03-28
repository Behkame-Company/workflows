# ReAct Pattern

> Reasoning + Acting interleaved in a loop — the core pattern behind most modern coding agents, grounding each decision in real observations from the environment.

---

## The Core Loop

ReAct (**Re**asoning + **Act**ing) interleaves thinking and doing in a tight loop. The agent:

1. **Thinks** about what to do (Reason)
2. **Takes an action** using a tool (Act)
3. **Observes** the result (Observe)
4. **Reasons again** based on the new information

This cycle repeats until the task is complete.

```
┌──────────────────────────────────────────────────────────────────┐
│                       ReAct LOOP                                  │
│                                                                  │
│                   ┌──────────────┐                               │
│                   │   THOUGHT    │                               │
│              ┌───►│              │                               │
│              │    │  "I need to  │                               │
│              │    │   find the   │                               │
│              │    │   auth       │                               │
│              │    │   handler"   │                               │
│              │    └──────┬───────┘                               │
│              │           │                                        │
│              │           ▼                                        │
│              │    ┌──────────────┐                               │
│              │    │   ACTION     │                               │
│              │    │              │                               │
│              │    │  grep -r     │                               │
│              │    │  "authHandler"│                               │
│              │    │  src/        │                               │
│              │    └──────┬───────┘                               │
│              │           │                                        │
│              │           ▼                                        │
│              │    ┌──────────────┐                               │
│              │    │ OBSERVATION  │                               │
│              │    │              │                               │
│              │    │ Found in     │         ┌────────────┐       │
│              │    │ src/auth.ts  │────────►│  Task      │       │
│              │    │ line 42      │  done?  │  Complete  │       │
│              │    └──────┬───────┘   yes   └────────────┘       │
│              │           │ no                                     │
│              └───────────┘                                        │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

## Why ReAct Works

The key insight: **grounding reasoning in real observations prevents hallucination**.

Without ReAct (pure reasoning):
```
Agent: "The auth handler is probably in src/middleware/auth.js based on common patterns"
       → Makes changes to a file that doesn't exist
       → Hallucinates the file structure
```

With ReAct (grounded reasoning):
```
Agent: "I need to find the auth handler" (thought)
       → grep for auth handler (action)
       → Found it in src/routes/auth.ts:42 (observation)
       → "Now I know the exact location, I'll modify it" (thought)
       → Edit src/routes/auth.ts (action)
       → File updated successfully (observation)
```

## ReAct in Coding: A Full Trace

Here's what a real coding agent ReAct trace looks like:

```
┌──────────────────────────────────────────────────────────────────┐
│  Task: "Fix the login endpoint returning 500 errors"             │
│                                                                  │
│  THOUGHT 1: I should first reproduce the error by looking at     │
│  the logs or the endpoint code.                                  │
│                                                                  │
│  ACTION 1: grep -rn "login" src/routes/                          │
│                                                                  │
│  OBSERVATION 1:                                                  │
│    src/routes/auth.ts:15: router.post('/login', handleLogin)     │
│    src/routes/auth.ts:42: async function handleLogin(req, res)   │
│                                                                  │
│  THOUGHT 2: Found the handler. Let me read the function to       │
│  understand what it does and find the bug.                        │
│                                                                  │
│  ACTION 2: read_file("src/routes/auth.ts", lines 42-80)         │
│                                                                  │
│  OBSERVATION 2:                                                  │
│    async function handleLogin(req, res) {                        │
│      const user = await db.users.findOne({ email: req.body.email });
│      const valid = bcrypt.compare(req.body.password, user.hash); │
│      //  ^ missing 'await' — bcrypt.compare returns a Promise!   │
│                                                                  │
│  THOUGHT 3: Found it — bcrypt.compare is async but missing       │
│  await. This means 'valid' is always truthy (it's a Promise),    │
│  but later code may fail. Let me check what happens next.        │
│                                                                  │
│  ACTION 3: read_file("src/routes/auth.ts", lines 45-55)         │
│                                                                  │
│  OBSERVATION 3:                                                  │
│    if (!valid) { return res.status(401).json({...}) }            │
│    const token = jwt.sign({ id: user.id, role: user.role }, ...) │
│    // user.role is accessed but not selected from DB              │
│                                                                  │
│  THOUGHT 4: Two bugs: (1) missing await on bcrypt.compare,       │
│  (2) user.role may be undefined. Let me fix both.                │
│                                                                  │
│  ACTION 4: edit_file("src/routes/auth.ts", ...)                  │
│                                                                  │
│  OBSERVATION 4: File updated successfully                        │
│                                                                  │
│  THOUGHT 5: Now let me run the tests to verify.                  │
│                                                                  │
│  ACTION 5: npm test -- --grep "login"                            │
│                                                                  │
│  OBSERVATION 5: 4 tests passed, 0 failed ✓                      │
│                                                                  │
│  THOUGHT 6: Both bugs fixed, tests pass. Task complete.          │
└──────────────────────────────────────────────────────────────────┘
```

## Implementation

### Basic ReAct Loop

```typescript
interface ReActStep {
  thought: string;
  action: ToolCall;
  observation: string;
}

async function reactLoop(task: string, tools: Tool[]): Promise<string> {
  const history: ReActStep[] = [];
  const maxSteps = 20;

  for (let step = 0; step < maxSteps; step++) {
    // Ask the model to think and choose an action
    const response = await llm.complete({
      system: `You are a coding agent using the ReAct pattern.
               For each step:
               1. THOUGHT: Reason about what to do next based on observations
               2. ACTION: Choose a tool to call
               
               When the task is complete, respond with THOUGHT: Task complete.
               and ACTION: finish(summary)`,
      tools,
      prompt: `Task: ${task}
               
               Previous steps:
               ${history.map((s, i) => `
               Step ${i + 1}:
                 THOUGHT: ${s.thought}
                 ACTION: ${s.action.name}(${JSON.stringify(s.action.args)})
                 OBSERVATION: ${s.observation}
               `).join("\n")}
               
               What is your next thought and action?`
    });

    // Parse thought and action from response
    const { thought, action } = parseReActResponse(response);

    // Check for completion
    if (action.name === "finish") {
      return action.args.summary;
    }

    // Execute the action and observe the result
    const observation = await executeTool(action, tools);

    history.push({ thought, action, observation });
  }

  return "Max steps reached without completion";
}
```

### ReAct for Code Editing

```typescript
async function reactCodeEditor(issue: string) {
  const tools = [
    {
      name: "search",
      description: "Search for patterns in the codebase. Use to find relevant code.",
      execute: async (args: { pattern: string; path?: string }) => {
        return await shell(`grep -rn "${args.pattern}" ${args.path || "src/"}`);
      }
    },
    {
      name: "read_file",
      description: "Read a file or specific line range.",
      execute: async (args: { path: string; startLine?: number; endLine?: number }) => {
        return await readFileContents(args.path, args.startLine, args.endLine);
      }
    },
    {
      name: "edit_file",
      description: "Replace text in a file. Provide old_text and new_text.",
      execute: async (args: { path: string; oldText: string; newText: string }) => {
        return await replaceInFile(args.path, args.oldText, args.newText);
      }
    },
    {
      name: "run_tests",
      description: "Run the test suite. Returns pass/fail with error details.",
      execute: async (args: { filter?: string }) => {
        return await shell(`npm test ${args.filter ? `-- --grep "${args.filter}"` : ""}`);
      }
    },
    {
      name: "finish",
      description: "Declare the task complete with a summary of changes.",
      execute: async (args: { summary: string }) => args.summary
    }
  ];

  return await reactLoop(issue, tools);
}
```

### Python Implementation

```python
from dataclasses import dataclass
from typing import Callable

@dataclass
class ReActStep:
    thought: str
    action: str
    action_input: dict
    observation: str

def react_loop(task: str, tools: dict[str, Callable], max_steps: int = 20) -> str:
    history: list[ReActStep] = []

    for step in range(max_steps):
        # Build prompt with history
        history_text = "\n".join(
            f"Step {i+1}:\n"
            f"  Thought: {s.thought}\n"
            f"  Action: {s.action}({s.action_input})\n"
            f"  Observation: {s.observation}"
            for i, s in enumerate(history)
        )

        response = llm.complete(
            system="You are a ReAct coding agent. Think, then act.",
            prompt=f"Task: {task}\n\nHistory:\n{history_text}\n\nNext step:"
        )

        thought, action, action_input = parse_react(response)

        if action == "finish":
            return action_input["summary"]

        # Execute action and get observation
        tool_fn = tools[action]
        observation = tool_fn(**action_input)

        history.append(ReActStep(thought, action, action_input, observation))

    return "Max steps reached"
```

## ReAct vs. Other Patterns

```
┌────────────────────────────────────────────────────────────────┐
│                 PATTERN COMPARISON                               │
│                                                                │
│  PURE REASONING (Chain-of-Thought)                             │
│  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐                 │
│  │Think 1 │→│Think 2 │→│Think 3 │→│Answer  │                 │
│  └────────┘ └────────┘ └────────┘ └────────┘                 │
│  Problem: No grounding — can hallucinate facts                 │
│                                                                │
│  PURE ACTING (No reasoning)                                    │
│  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐                 │
│  │ Act 1  │→│ Act 2  │→│ Act 3  │→│ Act 4  │                 │
│  └────────┘ └────────┘ └────────┘ └────────┘                 │
│  Problem: No planning — flails between actions                 │
│                                                                │
│  ReAct (Interleaved)                                           │
│  ┌──────┐ ┌─────┐ ┌──────┐ ┌──────┐ ┌─────┐ ┌──────┐       │
│  │Think │→│ Act │→│ Obs  │→│Think │→│ Act │→│ Obs  │→...    │
│  └──────┘ └─────┘ └──────┘ └──────┘ └─────┘ └──────┘       │
│  Best of both: grounded reasoning + purposeful action          │
│                                                                │
│  PLAN-AND-EXECUTE (Separated)                                  │
│  ┌──────────────────┐ ┌──────────────────────────────┐        │
│  │  Plan all steps   │→│  Execute step by step         │        │
│  └──────────────────┘ └──────────────────────────────┘        │
│  More structured but less adaptive to surprises                │
└────────────────────────────────────────────────────────────────┘
```

| Aspect | ReAct | Chain-of-Thought | Plan-and-Execute |
|--------|-------|------------------|------------------|
| **Grounding** | High — every thought informed by real data | Low — reasoning without verification | Medium — plan may not survive contact with reality |
| **Adaptability** | High — re-reasons after each observation | N/A — single pass | Medium — can re-plan but less fluid |
| **Predictability** | Lower — path emerges during execution | High — single output | Higher — plan is visible upfront |
| **Best for** | Exploration, debugging, open-ended coding | Simple analysis, summarization | Complex multi-step with known structure |

## Common ReAct Pitfalls

### 1. Observation Overflow

When tool outputs are too large, they flood the context window:

```typescript
// Bad: dumping entire file contents
const observation = await readFile("src/index.ts"); // 2000 lines

// Better: targeted reads
const observation = await readFile("src/index.ts", { startLine: 42, endLine: 60 });
```

### 2. Thought Loops

The agent keeps reasoning about the same thing without making progress:

```typescript
// Detect thought loops
function isStuck(history: ReActStep[]): boolean {
  if (history.length < 3) return false;
  
  const lastThree = history.slice(-3);
  const thoughts = lastThree.map(s => s.thought);
  
  // If recent thoughts are too similar, the agent is stuck
  return similarity(thoughts[0], thoughts[2]) > 0.9;
}
```

### 3. Missing the Forest for the Trees

The agent gets lost in details and forgets the overall goal:

```typescript
// Periodically remind of the goal
const prompt = step % 5 === 0
  ? `Remember your overall goal: ${task}\n\nGiven your progress so far, what should you do next?`
  : `Based on the last observation, what's your next step?`;
```

## ReAct in Real Coding Agents

Every major coding agent uses ReAct at its core:

- **Copilot CLI**: Each tool call (read, edit, shell) is an action; the agent reasons between calls
- **Claude Code**: Interleaves thinking and tool use in its terminal loop
- **Cursor/Windsurf**: Agent tab uses ReAct to explore and modify code
- **Devin/OpenHands**: Multi-step coding tasks with observation-grounded reasoning

The explicit "Thought → Action → Observation" framing is what makes these tools work reliably on complex tasks.

## Next Steps

- [Autonomous Agents](./autonomous-agents.md) — full autonomous loop built on ReAct
- [Plan-and-Execute](./plan-and-execute.md) — separating planning from execution
- [Reflection and Self-Critique](./reflection.md) — adding self-evaluation to the loop
- [Shell and File Tools](../04-tool-use/shell-and-file-tools.md) — the tools agents act with
