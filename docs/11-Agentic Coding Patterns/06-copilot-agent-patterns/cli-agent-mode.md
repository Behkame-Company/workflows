# Copilot CLI Agent Mode

> Local autonomous coding with the Copilot CLI — interactive conversations or headless automation, with full access to your shell, file system, and tools.

---

## Two Interfaces

Copilot CLI offers two ways to run the agent:

```
┌──────────────────────────────────────────────────────────────┐
│                  Copilot CLI Interfaces                       │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│   Interactive                    Programmatic                │
│   ───────────                    ────────────                │
│   $ copilot                      $ copilot -p "task" \       │
│                                    --allow-all-tools         │
│   • Conversation flow                                        │
│   • Plan mode (Shift+Tab)        • Headless execution        │
│   • Steer mid-response           • CI/CD integration         │
│   • Approve/reject actions        • Script automation         │
│   • Full tool access              • Non-interactive           │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

## Interactive Mode

Launch with `copilot` in your terminal to start a conversation:

```bash
$ copilot
> Add input validation to the user registration endpoint

# Agent explores codebase, identifies files, proposes changes
# You approve or steer as it works
```

### Key Interactive Features

**Plan Mode (Shift+Tab):**
Toggle between coding and planning. In plan mode, the agent outlines its approach without making changes — useful for reviewing strategy before execution.

```
┌─────────────────────────────────┐
│         Plan Mode Flow          │
├─────────────────────────────────┤
│                                 │
│  Normal Mode ──Shift+Tab──▶     │
│                                 │
│     Plan Mode                   │
│     • Agent explains approach   │
│     • No files modified         │
│     • Review the strategy       │
│                                 │
│  ◀──Shift+Tab── Normal Mode     │
│     • Agent executes plan       │
│     • Files modified            │
│                                 │
└─────────────────────────────────┘
```

**Steering Mid-Response:**
While the agent is working, you can type to redirect it. The agent reads your input and adjusts its approach without starting over.

**Inline Feedback on Rejection:**
When you reject a proposed tool call, you can explain why. The agent uses your feedback to try a different approach.

### Essential Commands

| Command | Description |
|---|---|
| `/model` | Switch between available models |
| `/mcp` | Manage MCP server connections |
| `/allow-all` | Allow all tool calls without prompting |
| `/compact` | Manually compact context to free space |
| `/context` | Show token usage and context status |
| `/feedback` | Send feedback to the Copilot team |
| `/help` | Show all available commands |
| `/clear` | Clear conversation history |

## Programmatic Mode

Run the agent non-interactively for automation:

```bash
# Single task, all tools allowed
copilot -p "Add error handling to src/api/users.ts" --allow-all-tools

# Pipe output to another tool
copilot -p "Explain the authentication flow" | cat

# Use in a script
RESULT=$(copilot -p "What test framework does this project use?")
echo "$RESULT"
```

### Programmatic Flags

```bash
copilot -p "task description"    # Run non-interactively
  --allow-all-tools              # Skip all permission prompts
  --allow-tool "Edit"            # Allow specific tool
  --deny-tool "Shell"            # Deny specific tool
  --model "claude-sonnet-4"      # Select model
  --quiet                        # Suppress non-essential output
```

## Context Management

The agent works within a finite context window. Understanding context management is critical for long sessions.

```
┌─────────────────────────────────────────────────┐
│              Context Window                      │
├─────────────────────────────────────────────────┤
│                                                  │
│  ┌──────────────────────────────────────┐        │
│  │  System prompt + Instructions        │  ~5%   │
│  ├──────────────────────────────────────┤        │
│  │  Conversation history                │        │
│  │  + Tool calls & results              │ ~90%   │
│  │  + File contents read                │        │
│  ├──────────────────────────────────────┤        │
│  │  Current response generation         │  ~5%   │
│  └──────────────────────────────────────┘        │
│                                                  │
│  At 95% capacity ──▶ Auto-compaction triggers    │
│  /compact ──▶ Manual compaction anytime          │
│  /context ──▶ Check current token usage          │
│                                                  │
└─────────────────────────────────────────────────┘
```

**Auto-compaction** triggers at ~95% context usage. The agent summarizes the conversation so far and continues with a compressed history.

**Manual compaction** with `/compact` lets you proactively free context space. Use this before starting a new phase of work.

**Context usage** with `/context` shows how many tokens you've used and how much capacity remains.

### Context Best Practices

- Use `/compact` between distinct phases of work
- Break large tasks into multiple sessions
- Let the agent write findings to files rather than holding everything in memory
- Use specific file paths instead of asking the agent to search broadly

## Tool Access and Permissions

The agent has access to a comprehensive toolset:

```
┌──────────────────────────────────────────────────┐
│              Agent Tool Access                    │
├──────────────────────────────────────────────────┤
│                                                   │
│  File System        Shell              Git        │
│  ───────────        ─────              ───        │
│  • Read files       • Run commands     • Status   │
│  • Edit files       • Install deps     • Diff     │
│  • Create files     • Build project    • Commit   │
│  • Search/glob      • Run tests        • Branch   │
│                     • Any CLI tool      • Log      │
│                                                   │
│  MCP Tools          Code Intelligence             │
│  ─────────          ─────────────────             │
│  • Custom servers   • Grep/search                 │
│  • External APIs    • Symbol lookup               │
│  • Databases        • File patterns               │
│  • Any MCP tool                                   │
│                                                   │
└──────────────────────────────────────────────────┘
```

### Permission Model

Control which tools the agent can use:

```bash
# Allow everything (for trusted tasks)
copilot --allow-all-tools

# Allow specific tools only
copilot --allow-tool "Edit" --allow-tool "Read"

# Deny specific tools
copilot --deny-tool "Shell"

# Interactive: approve each tool call individually (default)
copilot
```

In interactive mode, the agent asks permission before each tool call. You can:
- **Approve** — let it proceed
- **Reject with feedback** — explain why and suggest alternatives
- **Allow for session** — approve this tool type for the rest of the session

## Example Workflows

### Local Feature Implementation

```bash
$ copilot
> Implement a rate limiter middleware for Express.
> It should use a sliding window algorithm with Redis.
> The config should come from environment variables.
> Add tests using our existing Jest setup.

# Agent:
# 1. Explores existing middleware patterns in the codebase
# 2. Reads package.json for dependencies
# 3. Creates src/middleware/rateLimiter.ts
# 4. Creates src/middleware/__tests__/rateLimiter.test.ts
# 5. Updates src/middleware/index.ts exports
# 6. Runs npm test to verify
```

### PR Review Workflow

```bash
# Review a PR programmatically
copilot -p "Review the changes in the current branch compared to main. \
Focus on: security issues, error handling gaps, and missing tests. \
Output a structured review." --allow-all-tools
```

### Codebase Exploration

```bash
$ copilot
> I'm new to this codebase. Walk me through:
> 1. The overall architecture
> 2. How requests flow from API to database
> 3. Where business logic lives
> 4. How tests are organized
```

### Automated Refactoring

```bash
# Refactor all callback-style async to async/await
copilot -p "Find all files in src/ using callback-style async patterns \
and refactor them to use async/await. Run tests after each file change \
to ensure nothing breaks." --allow-all-tools
```

## Integration with MCP

Extend the agent's capabilities by connecting MCP servers:

```bash
# In interactive mode
/mcp

# Or configure in settings for persistent connections
# The agent can then use MCP tools alongside built-in tools
```

MCP servers give the agent access to databases, APIs, documentation servers, and custom tools — all through a standardized protocol.

## Tips for Effective Agent Sessions

1. **Be specific** — "Add input validation to `POST /api/users` in `src/routes/users.ts`" beats "add validation"
2. **Provide context** — mention relevant files, patterns, or constraints upfront
3. **Use plan mode first** — review the agent's strategy before it starts coding
4. **Compact between tasks** — keep context fresh for each new objective
5. **Leverage git** — the agent can commit checkpoints, making rollback easy
6. **Chain related work** — complete one feature fully before moving to the next
