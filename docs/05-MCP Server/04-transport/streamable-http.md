# Streamable HTTP Transport

> The modern, recommended transport for remote MCP servers — standard HTTP with optional streaming.

---

## How It Works

```
┌──────────────┐    POST /mcp     ┌──────────────┐
│              │ ──────────────→  │              │
│    Client    │                  │    Server    │
│              │ ←──────────────  │   (HTTP)     │
│              │   JSON or SSE    │              │
└──────────────┘                  └──────────────┘
```

1. Server exposes a **single HTTP endpoint** (e.g., `/mcp`)
2. Client sends each MCP message as an **HTTP POST**
3. Server responds with **JSON** (simple) or **SSE stream** (when streaming is needed)
4. Both client and server declare `Accept: application/json, text/event-stream`

---

## Why Streamable HTTP Replaced SSE

| Old (SSE) | New (Streamable HTTP) |
|-----------|----------------------|
| Two endpoints (POST + SSE stream) | Single endpoint |
| Persistent connection required | Stateless by default |
| Proxy/firewall issues | Standard HTTP, proxy-friendly |
| Complex session management | Simple request-response |
| Unidirectional streaming only | Bidirectional when needed |

---

## Request/Response Patterns

### Simple Request-Response
```http
POST /mcp HTTP/1.1
Content-Type: application/json
Accept: application/json, text/event-stream

{"jsonrpc":"2.0","id":1,"method":"tools/list"}
```

```http
HTTP/1.1 200 OK
Content-Type: application/json

{"jsonrpc":"2.0","id":1,"result":{"tools":[...]}}
```

### Streaming Response
When the server needs to send progress updates or multiple results:

```http
POST /mcp HTTP/1.1
Content-Type: application/json
Accept: application/json, text/event-stream

{"jsonrpc":"2.0","id":5,"method":"tools/call","params":{"name":"long_running_task"}}
```

```http
HTTP/1.1 200 OK
Content-Type: text/event-stream

data: {"jsonrpc":"2.0","method":"notifications/progress","params":{"progress":25}}

data: {"jsonrpc":"2.0","method":"notifications/progress","params":{"progress":75}}

data: {"jsonrpc":"2.0","id":5,"result":{"content":[{"type":"text","text":"Done!"}]}}
```

---

## Session Management

Streamable HTTP supports optional session tracking via the `Mcp-Session-Id` header:

```http
# Server includes session ID in initialization response
Mcp-Session-Id: abc-123-def

# Client includes it in subsequent requests
POST /mcp HTTP/1.1
Mcp-Session-Id: abc-123-def
```

This enables stateful sessions over stateless HTTP when needed.

---

## Security Requirements

| Requirement | Details |
|-------------|---------|
| **TLS** | Always use HTTPS for remote servers |
| **Origin validation** | Server must check the `Origin` header |
| **Authentication** | OAuth 2.1 with PKCE for production deployments |
| **Localhost binding** | Local HTTP servers must bind to `127.0.0.1`, not `0.0.0.0` |
| **DNS rebinding** | Validate Host header to prevent DNS rebinding attacks |

---

## Implementation

### Python
```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("my-remote-server")

@mcp.tool
async def query_data(sql: str) -> str:
    """Run a read-only SQL query."""
    # ... execute query
    return results

if __name__ == "__main__":
    mcp.run(transport="streamable-http", host="0.0.0.0", port=8080)
```

### TypeScript
```typescript
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StreamableHTTPServerTransport } from "@modelcontextprotocol/sdk/server/streamablehttp.js";

const server = new Server({ name: "my-server", version: "1.0.0" });
// Register tools...

const transport = new StreamableHTTPServerTransport({ endpoint: "/mcp" });
await server.connect(transport);
// Serve on port 8080
```

---

## When to Use Streamable HTTP

See [Transport Overview](transport-overview.md) for a comparison of all transport types.

**Use Streamable HTTP for**: remote/cloud deployments, multiple clients, corporate proxies, team-shared servers.

**Use stdio instead for**: local single-user tools or when maximum latency performance is needed.

---

## Key Takeaways

1. **Single endpoint** — one URL handles all MCP communication
2. **Standard HTTP** — works with existing proxies, load balancers, and firewalls
3. **Optional streaming** — SSE only when the server needs to stream
4. **Always use TLS** — never expose unencrypted MCP over the network
5. **Replaced SSE transport** — migrate legacy SSE servers to Streamable HTTP
