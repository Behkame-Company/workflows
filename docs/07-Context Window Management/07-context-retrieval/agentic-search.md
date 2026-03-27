# Agentic Search

> When AI agents autonomously discover and retrieve the context they need.

---

## What Is Agentic Search?

Agentic search is when the AI agent, rather than the developer, decides what context to retrieve and when. The agent uses tools to explore, search, and load information as its task requires.

```
Developer provides: "Fix the login bug"

Agent autonomously:
  1. Searches codebase for "login" → finds relevant files
  2. Reads error logs → identifies the symptom
  3. Reads src/auth/login.ts → finds the bug
  4. Reads tests/auth.test.ts → understands expected behavior
  5. Fixes the bug
  6. Runs tests → verifies the fix
```

The developer never told the agent which files to read. The agent discovered them.

---

## How Agents Navigate Information

### Using Metadata as Signals
Agents learn from the environment without reading everything:
- **File names** suggest purpose (`test_utils.py` → testing utility)
- **Directory structure** implies organization (`src/auth/` → auth module)
- **File sizes** indicate complexity (large file → may need analysis)
- **Timestamps** can proxy for relevance (recently modified → active)
- **Naming conventions** hint at relationships (`user.model.ts` → `user.service.ts`)

### Progressive Discovery
```
Step 1: ls src/           → See module structure
Step 2: ls src/auth/      → See auth files
Step 3: grep "login" src/auth/  → Find relevant code
Step 4: Read specific function  → Understand the issue
```

Each step narrows the search, consuming minimal tokens.

---

## Tools for Agentic Search

| Tool | Purpose | Token Cost |
|------|---------|-----------|
| **glob** | Find files by pattern | Very low (~50-200 tokens) |
| **grep** | Search content by pattern | Low (~100-500 tokens) |
| **ls/dir** | List directory contents | Very low (~50-200 tokens) |
| **head/tail** | Preview file start/end | Low (~100-300 tokens) |
| **git log** | Recent changes | Low (~200-500 tokens) |
| **read (targeted)** | Read specific lines | Medium (~200-1,000 tokens) |
| **read (full file)** | Read entire file | High (~500-3,000 tokens) |

**Best practice**: Cheapest tools first, then progressively more expensive.

---

## Prompting for Agentic Search

```markdown
## Exploration Strategy
When investigating an issue or implementing a feature:
1. Start with glob/grep to find relevant files (cheap)
2. Read file headers or exports to understand structure
3. Only read full files when you need implementation details
4. Use git log to see recent changes to relevant areas
5. Prefer targeted reads (specific lines) over full file reads
```

---

## Key Takeaways

1. **Agents can self-navigate** — Given good tools, they find what they need
2. **Cheap tools first** — glob/grep before file reads
3. **Metadata is free context** — File names, sizes, timestamps
4. **Progressive narrowing** — Each step reduces the search space
5. **Prompt for exploration strategy** — Guide HOW the agent searches

---

## Next Steps

- 🔗 [Progressive Disclosure](progressive-disclosure.md) — Layer-by-layer context
- 🔗 [Context Isolation with Sub-Agents](../08-sub-agent-architectures/context-isolation-with-subagents.md) — Parallel search
