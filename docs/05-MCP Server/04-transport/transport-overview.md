# Transport Overview

> How MCP messages travel between client and server — local, remote, and everything in between.

---

## What Is a Transport?

The transport layer handles **how** JSON-RPC messages are delivered between MCP clients and servers. The protocol content is the same regardless of transport — only the delivery mechanism changes.

```
MCP Protocol (JSON-RPC messages) — WHAT is sent
Transport Layer                  — HOW it's delivered
```

---

## Available Transports

| Transport | Use Case | Network? | Status |
|-----------|----------|----------|--------|
| **stdio** | Local tools, CLI scripts, desktop apps | No | ✅ Current |
| **Streamable HTTP** | Remote servers, cloud services, multi-client | Yes | ✅ Current (recommended) |
| **SSE (HTTP + Server-Sent Events)** | Remote servers (legacy) | Yes | ⚠️ Deprecated |

---

## Choosing a Transport

```
Decision Tree:
├── Server runs on same machine as client?
│   ├── Yes → stdio (simplest, fastest, most secure)
│   └── No  → Streamable HTTP
├── Multiple clients need to connect?
│   ├── Yes → Streamable HTTP
│   └── No  → Either works
├── Going through corporate proxy/firewall?
│   └── Streamable HTTP (standard HTTP, proxy-friendly)
└── Maximum security needed?
    └── stdio (no network exposure at all)
```

---

## Transport Comparison

| Feature | stdio | Streamable HTTP | SSE (Deprecated) |
|---------|-------|----------------|-------------------|
| **Latency** | Lowest (in-process) | Network RTT | Network RTT |
| **Setup complexity** | Minimal | HTTP server needed | Complex (2 endpoints) |
| **Network exposure** | None | Yes (requires TLS) | Yes |
| **Multi-client** | No (1:1) | Yes | Limited |
| **Firewall-friendly** | N/A | Yes | Problematic |
| **Streaming** | Inherent (pipe) | SSE within HTTP | One-way only |
| **Authentication** | Process-level | OAuth 2.1 | OAuth |
| **Best for** | Local tools, desktop | Cloud, teams, enterprise | Legacy (migrate away) |

---

## How Messages Look on the Wire

### stdio
Messages are newline-delimited JSON on stdin/stdout:
```
→ stdin:  {"jsonrpc":"2.0","id":1,"method":"tools/list"}\n
← stdout: {"jsonrpc":"2.0","id":1,"result":{"tools":[...]}}\n
```

### Streamable HTTP
Messages are standard HTTP POST/response:
```
→ POST /mcp HTTP/1.1
  Content-Type: application/json
  {"jsonrpc":"2.0","id":1,"method":"tools/list"}

← 200 OK
  Content-Type: application/json
  {"jsonrpc":"2.0","id":1,"result":{"tools":[...]}}
```

For streaming responses:
```
← 200 OK
  Content-Type: text/event-stream
  data: {"jsonrpc":"2.0","method":"notifications/progress","params":{"progress":50}}
  data: {"jsonrpc":"2.0","id":1,"result":{...}}
```

---

## Key Principles

1. **Protocol content is transport-agnostic** — Same JSON-RPC messages regardless of transport
2. **One transport per connection** — Don't mix transports within a session
3. **stdio for local, HTTP for remote** — Simple rule that covers 99% of cases
4. **Always use TLS for HTTP** — Never expose MCP servers over unencrypted HTTP
5. **Custom transports are allowed** — As long as they follow JSON-RPC lifecycle rules

---

## Next Steps

- 🔗 [STDIO Transport](stdio.md) — Local subprocess communication
- 🔗 [Streamable HTTP](streamable-http.md) — The modern remote transport
- 🔗 [SSE (Deprecated)](sse-deprecated.md) — Migration guide
