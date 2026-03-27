# STDIO Transport

> The simplest MCP transport — your server runs as a subprocess, communicating via stdin and stdout.

---

## How It Works

```
┌──────────────┐       stdin        ┌──────────────┐
│              │ ──────────────────→ │              │
│    Client    │                     │    Server    │
│   (Host)     │ ←────────────────── │  (subprocess)│
│              │       stdout        │              │
└──────────────┘                     └──────────────┘
                    stderr → logs (optional)
```

1. The **host** launches the server as a **child process**
2. The client writes JSON-RPC messages to the server's **stdin**
3. The server writes JSON-RPC responses to **stdout**
4. **stderr** is available for logging/debugging (not MCP messages)
5. When the session ends, the process is terminated

---

## Configuration

### Claude Desktop
```json
{
  "mcpServers": {
    "my-server": {
      "command": "python",
      "args": ["server.py"],
      "env": { "API_KEY": "secret" }
    }
  }
}
```

### VS Code
```json
{
  "mcp": {
    "servers": {
      "my-server": {
        "command": "npx",
        "args": ["my-mcp-server"],
        "env": { "API_KEY": "secret" }
      }
    }
  }
}
```

### Common Patterns
```json
// Python server
{ "command": "python", "args": ["-m", "my_mcp_server"] }
{ "command": "uv", "args": ["run", "my-server"] }

// Node/TypeScript server
{ "command": "npx", "args": ["@my-org/mcp-server"] }
{ "command": "node", "args": ["dist/server.js"] }

// Binary server
{ "command": "/usr/local/bin/my-mcp-server" }
```

---

## Message Format

Messages are **newline-delimited JSON**. Each message is a complete JSON object on a single line:

```
Client → stdin:
{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2025-03-26","capabilities":{}}}\n

Server → stdout:
{"jsonrpc":"2.0","id":1,"result":{"protocolVersion":"2025-03-26","capabilities":{"tools":{}}}}\n
```

### Rules
- One JSON object per line
- No embedded newlines within messages
- UTF-8 encoding
- **Nothing** except MCP messages on stdout (no print statements!)
- Use stderr for logging

---

## Implementation

### Python (FastMCP)
```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("my-server")

@mcp.tool
async def hello(name: str) -> str:
    """Greet the user by name."""
    return f"Hello, {name}!"

if __name__ == "__main__":
    mcp.run()  # Defaults to stdio transport
```

### TypeScript
```typescript
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";

const server = new Server({
  name: "my-server",
  version: "1.0.0"
}, {
  capabilities: { tools: {} }
});

// Register tools...

const transport = new StdioServerTransport();
await server.connect(transport);
```

---

## Advantages

| Advantage | Why It Matters |
|-----------|---------------|
| **Zero network exposure** | No ports open, no attack surface |
| **Lowest latency** | In-process pipe, no network RTT |
| **Simplest setup** | Just a command + args, no HTTP server |
| **Process isolation** | OS-level security boundary |
| **Easy debugging** | stderr for logs, process lifecycle managed by host |

---

## Limitations

| Limitation | Workaround |
|-----------|-----------|
| **Single client only** | Use Streamable HTTP for multi-client |
| **Local only** | Use Streamable HTTP for remote |
| **No reconnection** | Process restarts start fresh |
| **stdout pollution** | Never use `print()` — use stderr or MCP logging |

---

## Common Pitfalls

### ❌ Printing to stdout
```python
# BAD — This breaks the MCP protocol!
print("Debug: processing request")

# GOOD — Use stderr
import sys
print("Debug: processing request", file=sys.stderr)
```

### ❌ Multi-line JSON
```
# BAD
{
  "jsonrpc": "2.0",
  "id": 1
}

# GOOD
{"jsonrpc":"2.0","id":1}
```

---

## Key Takeaways

1. stdio is the **default and simplest** transport for local MCP servers
2. Server runs as a **subprocess** of the host
3. **Never** write non-MCP content to stdout — use stderr for logs
4. Messages are **newline-delimited JSON** on stdin/stdout
5. Best for **local tools** — use Streamable HTTP for remote/multi-client

---

## Next Steps

- 🔗 [Streamable HTTP](streamable-http.md) — For remote and multi-client scenarios
- 🔗 [Your First Server (Python)](../05-building-servers/first-server-python.md) — Build a stdio server
