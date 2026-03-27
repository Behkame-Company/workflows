# The Client-Host-Server Model

> MCP's three-layer architecture — understanding who does what and why separation matters.

---

## Architecture Overview

```
┌───────────────────────────────────────────────────┐
│                      HOST                          │
│  (Claude Desktop, VS Code, Cursor, your app)       │
│                                                     │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐│
│  │   Client A   │  │   Client B   │  │   Client C   ││
│  │  (1:1 with   │  │  (1:1 with   │  │  (1:1 with   ││
│  │   Server A)  │  │   Server B)  │  │   Server C)  ││
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘│
│         │                │                │         │
│         │      AI/LLM Integration         │         │
│         │      Permission Enforcement     │         │
│         │      Context Aggregation        │         │
└─────────┼────────────────┼────────────────┼─────────┘
          │                │                │
    ┌─────┴─────┐   ┌─────┴─────┐   ┌─────┴─────┐
    │  Server A  │   │  Server B  │   │  Server C  │
    │  (GitHub)  │   │  (Postgres) │   │ (Filesystem)│
    │            │   │            │   │            │
    │  Tools:    │   │  Tools:    │   │  Tools:    │
    │  - issues  │   │  - query   │   │  - read    │
    │  - PRs     │   │  - schema  │   │  - write   │
    │  - search  │   │            │   │  - search  │
    └───────────┘   └───────────┘   └───────────┘
```

---

## The Three Layers

### 1. Host

The **host** is the application the developer uses. It:

- **Creates and manages** MCP clients (one per connected server)
- **Integrates with the AI/LLM** — sends prompts, receives completions
- **Enforces security** — permissions, user consent, trust boundaries
- **Aggregates context** — combines data from multiple servers into coherent AI context

**Examples**: Claude Desktop, VS Code with Copilot, Cursor, custom agent applications

**Key principle**: The host is the **trust anchor**. It sees everything and controls what the AI can access.

### 2. Client

The **client** is the protocol handler inside the host. It:

- Maintains a **1:1 stateful session** with exactly one MCP server
- Handles **capability negotiation** during initialization
- Routes **requests, responses, and notifications** between host and server
- Maintains **isolation** — clients don't share state with each other

**Key principle**: Each client is an independent, isolated connection. Server A cannot see Server B's data through the client layer.

### 3. Server

The **server** is the external program that exposes capabilities. It:

- **Declares its capabilities** during initialization (which primitives it supports)
- **Exposes tools** (actions), **resources** (data), and **prompts** (templates)
- **Responds to requests** from the client
- **Can request sampling** — asking the AI through the client to reason about something

**Examples**: GitHub MCP server, Postgres MCP server, filesystem server, Sentry server

**Key principle**: Servers are **focused and composable**. Each server does one thing well.

---

## Information Flow

```
Developer types: "Show me recent errors from Sentry for the auth module"

Host (VS Code/Copilot):
  1. Receives user message
  2. Sends to AI/LLM with available tool definitions from all connected servers
  
AI/LLM:
  3. Decides to call Sentry server's "search_issues" tool
  4. Returns tool call request to host

Host:
  5. Validates the request (is this tool allowed? does user consent?)
  6. Routes to Client C (connected to Sentry server)

Client C:
  7. Sends JSON-RPC request to Sentry server

Sentry Server:
  8. Queries Sentry API
  9. Returns structured results

Client C → Host → AI/LLM:
  10. AI formats the results for the developer
  
Host:
  11. Displays the response to the developer
```

---

## Why Three Layers?

### Security Isolation

```
❌ Without separation:
  AI ←→ GitHub (AI can do anything to your repos)

✅ With MCP's three layers:
  AI ←→ Host (enforces permissions) ←→ Client (isolated) ←→ Server (focused)
```

The host **mediates every interaction**, preventing:
- The AI from accessing tools without user consent
- One server from seeing another server's data
- The AI from accumulating implicit permissions over time

### Composability

Each server is independent. You can:
- Add a new server without changing existing ones
- Remove a server without breaking anything
- Replace a server with a different implementation
- Run servers locally (stdio) or remotely (HTTP) — the host handles both

### Scalability

```
Small team:  Host + 2-3 servers (GitHub, filesystem, database)
Medium team: Host + 5-10 servers + shared configuration
Enterprise:  Host + Gateway + dozens of servers + multi-tenant routing
```

---

## Real-World Configuration

### Claude Desktop (claude_desktop_config.json)
```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["@anthropic-ai/mcp-server-github"],
      "env": { "GITHUB_TOKEN": "ghp_..." }
    },
    "postgres": {
      "command": "npx",
      "args": ["@anthropic-ai/mcp-server-postgres", "postgresql://localhost/mydb"]
    },
    "filesystem": {
      "command": "npx",
      "args": ["@anthropic-ai/mcp-server-filesystem", "/home/user/projects"]
    }
  }
}
```

### VS Code (settings.json)
```json
{
  "mcp": {
    "servers": {
      "github": {
        "command": "npx",
        "args": ["@anthropic-ai/mcp-server-github"],
        "env": { "GITHUB_TOKEN": "ghp_..." }
      }
    }
  }
}
```

---

## Key Takeaways

1. **Host** = your IDE/app. Controls everything, enforces security.
2. **Client** = protocol handler. 1:1 with a server, isolated from other clients.
3. **Server** = tool provider. Focused on one domain, exposes tools/resources/prompts.
4. **Three layers exist for security** — the host mediates every AI-tool interaction.
5. **Composability** — add/remove servers like plugins.

---

## Next Steps

- 🔗 [Protocol Lifecycle](protocol-lifecycle.md) — How sessions start, negotiate, and end
- 🔗 [Tools](../03-primitives/tools.md) — The most important primitive

---

*"The host sees everything. The server sees nothing it shouldn't. The client ensures isolation."*
