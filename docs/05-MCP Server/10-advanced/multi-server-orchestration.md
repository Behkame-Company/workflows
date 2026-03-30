# Multi-Server Orchestration

> Connecting multiple MCP servers to create powerful AI agent workflows.

---

## How Multi-Server Works

The host connects to multiple servers simultaneously. Each server is independent, but the AI can use tools from all of them in a single conversation:

```
┌──────────────────────────────────────────────┐
│  HOST (Claude / Copilot / Cursor)            │
│                                              │
│  Available tools:                            │
│  ├── github:  create_issue, search_code      │
│  ├── db:      query, list_tables             │
│  ├── sentry:  get_errors, search_errors      │
│  └── slack:   send_message, search_messages  │
│                                              │
│  AI can freely combine tools across servers  │
└──────────────────────────────────────────────┘
```

---

## Orchestration Patterns

### 1. Investigate → Fix → Report
```
User: "Fix the top production error"

AI workflow:
  1. sentry: get_errors(status="unresolved", sort="frequency")
  2. github: search_code(query=error_function_name)
  3. github: create_pull_request(title="Fix: ...", body="...")
  4. slack: send_message(channel="#engineering", text="Fixed #123...")
```

### 2. Design → Build → Test
```
User: "Implement the new user profile page"

AI workflow:
  1. figma: get_components(file="user-profile")
  2. github: get_file_contents(path="src/components/")
  3. codebase: search for existing patterns
  4. playwright: screenshot(url="/profile")  ← verify result
```

### 3. Monitor → Diagnose → Resolve
```
User: "Why is the app slow?"

AI workflow:
  1. sentry: get_errors(tag="performance")
  2. db: query("SELECT * FROM slow_query_log LIMIT 10")
  3. github: search_code(query="N+1 query")
  4. github: create_issue(title="Performance: N+1 query in ...")
```

---

## Configuration Example

```jsonc
// .vscode/mcp.json — Multiple servers configured together
{
  "servers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": { "GITHUB_TOKEN": "${env:GITHUB_TOKEN}" }
    },
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": { "DATABASE_URL": "${env:DATABASE_URL}" }
    },
    "sentry": {
      "command": "npx",
      "args": ["-y", "@sentry/mcp-server"],
      "env": { "SENTRY_AUTH_TOKEN": "${env:SENTRY_AUTH_TOKEN}" }
    },
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp"]
    }
  }
}
```

---

## Best Practices

| Practice | Why |
|----------|-----|
| **Limit total tools** | Across all servers, keep under 30-40 total tools |
| **No tool name conflicts** | If two servers have `search`, one will be shadowed |
| **Start with 2-3 servers** | Add more as needed; too many = confusion |
| **Match server to workflow** | Only connect servers relevant to your current task |
| **Document combinations** | Tell team which server combos work well together |

---

## Key Takeaways

1. **Servers are independent** — The AI combines them, not the servers
2. **Design for workflows** — Think about cross-server task sequences
3. **Keep total tool count manageable** — 30-40 max across all servers
4. **Avoid name collisions** — Each tool name must be unique across servers
5. **Document recommended sets** — Create server "presets" for common workflows
