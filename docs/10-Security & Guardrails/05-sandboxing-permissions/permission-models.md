# AI Agent Permission Models

> How different AI coding tools handle permissions — from fully interactive approval to blanket allowlists — and how to configure them for safety without sacrificing productivity.

---

## Why Permissions Matter

AI coding agents can **read files**, **write code**, **run shell commands**, and **interact with external services**. Without permission controls, a single prompt injection or misjudged request could:

- Delete critical files
- Push untested code to production
- Leak secrets through shell commands
- Install malicious dependencies

Permission models are your **first line of defense** — they ensure the AI can only do what you've explicitly authorized.

## Copilot CLI Permission Model

Copilot CLI uses an **ask-first** model by default. Every potentially destructive action requires approval.

### Default Behavior

```
┌──────────────────────────────────────────────────────┐
│              COPILOT CLI DEFAULT MODE                 │
│                                                      │
│  User: "Fix the bug in auth.js"                      │
│                                                      │
│  Copilot: I need to edit src/auth.js                 │
│  ┌─────────────────────────────────────────┐         │
│  │  Allow edit to src/auth.js?             │         │
│  │  [Yes] [No] [Yes, for rest of session]  │         │
│  └─────────────────────────────────────────┘         │
│                                                      │
│  Copilot: I need to run: npm test                    │
│  ┌─────────────────────────────────────────┐         │
│  │  Allow shell command: npm test?         │         │
│  │  [Yes] [No] [Yes, for rest of session]  │         │
│  └─────────────────────────────────────────┘         │
└──────────────────────────────────────────────────────┘
```

### Allowing Specific Tools

Use `--allow-tool` to pre-approve specific operations:

```bash
# Allow a specific shell command
copilot-cli --allow-tool='shell(npm test)'

# Allow file editing
copilot-cli --allow-tool='write'

# Allow a specific MCP server tool
copilot-cli --allow-tool='MCP_SERVER_NAME(tool_name)'

# Allow multiple tools
copilot-cli \
  --allow-tool='shell(npm test)' \
  --allow-tool='shell(npm run lint)' \
  --allow-tool='write'
```

### Denying Specific Tools

Use `--deny-tool` to explicitly block dangerous operations:

```bash
# Block file deletion
copilot-cli --deny-tool='shell(rm)'

# Block pushing to remote
copilot-cli --deny-tool='shell(git push)'

# Block multiple dangerous commands
copilot-cli \
  --deny-tool='shell(rm)' \
  --deny-tool='shell(git push)' \
  --deny-tool='shell(curl)'
```

### Combining Allow and Deny

The most practical pattern: allow everything **except** dangerous operations.

```bash
# Allow all tools but block destructive commands
copilot-cli \
  --allow-all-tools \
  --deny-tool='shell(rm)' \
  --deny-tool='shell(git push)' \
  --deny-tool='shell(git checkout main)' \
  --deny-tool='shell(docker rm)' \
  --deny-tool='shell(kubectl delete)'
```

> **Deny takes precedence over allow.** If a tool matches both `--allow-tool` and `--deny-tool`, it is **denied**.

### Session-Level Approval

When Copilot prompts for permission, you have three options:

| Option | Effect |
|--------|--------|
| **Yes** | Allow this one action |
| **No** | Deny this action |
| **Yes, and approve for rest of session** | Allow this type of action for the entire session |

Session-level approval is a good middle ground — it reduces prompt fatigue while keeping you in the loop for the first occurrence.

### The --allow-all-tools Flag

```bash
# ⚠️ DANGER: Allows everything without asking
copilot-cli --allow-all-tools
```

```
┌──────────────────────────────────────────────────────┐
│  ⚠️  WARNING: --allow-all-tools                      │
│                                                      │
│  This flag disables ALL permission prompts.          │
│  The AI can:                                         │
│    • Edit any file in scope                          │
│    • Run any shell command                           │
│    • Delete files                                    │
│    • Push code to remotes                            │
│    • Access any MCP server tool                      │
│                                                      │
│  Use ONLY when:                                      │
│    • Working in a sandboxed/disposable environment   │
│    • Running on trusted code you fully control       │
│    • Combined with --deny-tool for dangerous ops     │
│                                                      │
│  NEVER use on:                                       │
│    • Untrusted repositories                          │
│    • Production environments                         │
│    • Shared machines                                 │
└──────────────────────────────────────────────────────┘
```

## Claude Code Permission Model

Claude Code uses a **sandboxed execution model** with a different approach to permissions:

```
┌──────────────────────────────────────────────────────┐
│            CLAUDE CODE PERMISSION MODEL               │
│                                                      │
│  File Operations:                                    │
│    Read    → Allowed (within project)                │
│    Write   → Requires approval (first time)          │
│    Delete  → Requires approval                       │
│                                                      │
│  Shell Commands:                                     │
│    Allowed list  → Runs without prompting            │
│    Other commands → Requires approval                │
│                                                      │
│  Configuration:                                      │
│    .claude/settings.json → Project-level settings    │
│    ~/.claude/settings.json → Global settings         │
└──────────────────────────────────────────────────────┘
```

```jsonc
// .claude/settings.json
{
  "permissions": {
    "allow": [
      "Bash(npm test)",
      "Bash(npm run lint)",
      "Bash(git status)",
      "Bash(git diff)"
    ],
    "deny": [
      "Bash(rm -rf)",
      "Bash(git push)",
      "Bash(curl)"
    ]
  }
}
```

## MCP Server Permissions

MCP servers introduce a **third-party trust surface**. Each server exposes tools the AI can call:

```bash
# Allow specific MCP server tools
copilot-cli --allow-tool='github(create_issue)'
copilot-cli --allow-tool='database(read_query)'

# Block MCP tools that modify data
copilot-cli --deny-tool='database(write_query)'
copilot-cli --deny-tool='github(merge_pull_request)'
```

### MCP Permission Decision Matrix

| MCP Server Type | Read Tools | Write Tools | Recommendation |
|----------------|------------|-------------|----------------|
| Documentation | ✅ Allow | N/A | Low risk |
| Database | ✅ Allow | ❌ Deny | Allow reads, block writes |
| Git/GitHub | ✅ Allow | ⚠️ Case-by-case | Allow status, deny push/merge |
| Deployment | ❌ Deny all | ❌ Deny all | Never auto-approve |
| File system | ⚠️ Scope | ⚠️ Scope | Limit to project directory |

## Permission Decision Flowchart

```
┌─────────────────────────────────┐
│  AI wants to perform an action  │
└──────────────┬──────────────────┘
               │
       ┌───────▼───────┐
       │ Is action on   │──── Yes ──→ ❌ BLOCK
       │ deny list?     │
       └───────┬───────┘
               │ No
       ┌───────▼───────┐
       │ Is action on   │──── Yes ──→ ✅ ALLOW
       │ allow list?    │
       └───────┬───────┘
               │ No
       ┌───────▼───────┐
       │ --allow-all-   │──── Yes ──→ ✅ ALLOW
       │ tools set?     │
       └───────┬───────┘
               │ No
       ┌───────▼───────┐
       │ Session-level  │──── Yes ──→ ✅ ALLOW
       │ approved?      │
       └───────┬───────┘
               │ No
       ┌───────▼───────┐
       │  PROMPT USER   │
       │  for approval  │
       └───────────────┘
```

## Practical Configuration Examples

### Frontend Development

```bash
copilot-cli \
  --allow-tool='write' \
  --allow-tool='shell(npm test)' \
  --allow-tool='shell(npm run lint)' \
  --allow-tool='shell(npm run build)' \
  --allow-tool='shell(npx)' \
  --deny-tool='shell(rm)' \
  --deny-tool='shell(git push)' \
  --deny-tool='shell(npm publish)'
```

### Backend Development

```bash
copilot-cli \
  --allow-tool='write' \
  --allow-tool='shell(python)' \
  --allow-tool='shell(pip install)' \
  --allow-tool='shell(pytest)' \
  --deny-tool='shell(rm)' \
  --deny-tool='shell(git push)' \
  --deny-tool='shell(curl)' \
  --deny-tool='shell(wget)'
```

### Code Review (Read-Only)

```bash
copilot-cli \
  --allow-tool='shell(git diff)' \
  --allow-tool='shell(git log)' \
  --allow-tool='shell(git show)' \
  --deny-tool='write' \
  --deny-tool='shell(git commit)' \
  --deny-tool='shell(git push)'
```

## Tips

✅ **Do:**
- Start with the **default interactive mode** and learn what permissions your workflow needs
- Use `--allow-tool` for specific commands you've verified are safe
- Combine `--allow-all-tools` with `--deny-tool` for a practical balance
- Use session-level approval for trusted, repetitive workflows
- Review MCP server tool descriptions before allowing them

❌ **Don't:**
- Use `--allow-all-tools` without any `--deny-tool` restrictions
- Allow all MCP server tools without reviewing their capabilities
- Grant write access when the task only requires reading or analysis
- Skip permission prompts without reading what action is being authorized
- Assume permission models prevent all attacks — they're one layer of defense

---

## Next Steps

- [Sandboxed Execution for AI Agents](./sandboxed-execution.md) — Add isolation when permissions alone aren't enough
- [Least Privilege for AI Agents](./least-privilege.md) — Systematic approach to minimal permissions
- [Network Access Control](./network-access.md) — Control what the AI can reach on the network
- [LLM08: Excessive Agency](../06-owasp-llm-top-10/llm08-excessive-agency.md) — Understand the OWASP risk of over-permissioned agents
