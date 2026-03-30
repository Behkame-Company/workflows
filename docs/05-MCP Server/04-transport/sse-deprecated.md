# SSE Transport (Deprecated)

> **⚠️ Deprecated**: SSE transport has been replaced by Streamable HTTP. See [Streamable HTTP](streamable-http.md) for the current approach.

> The original HTTP-based MCP transport. Replaced by Streamable HTTP as of March 2025.

---

## ⚠️ Deprecation Notice

**SSE transport was deprecated in the March 2025 MCP specification update.** All new servers should use Streamable HTTP. Existing SSE servers should migrate.

---

## How SSE Worked

```
┌──────────────┐    POST /message    ┌──────────────┐
│              │ ──────────────────→  │              │
│    Client    │                      │    Server    │
│              │ ←──────────────────  │              │
│              │    GET /sse (stream) │              │
└──────────────┘                      └──────────────┘
```

SSE required **two separate endpoints**:
1. **POST endpoint** (`/message`) — Client sends requests to server
2. **SSE endpoint** (`/sse`) — Server streams responses and notifications to client

---

## Why SSE Was Replaced

See [Transport Overview](transport-overview.md) for a comparison of all transport types.

SSE required two separate endpoints, a persistent connection (causing proxy/firewall issues), only supported unidirectional streaming, and had complex session management across two connections.

---

## Migration Guide: SSE → Streamable HTTP

### 1. Replace Two Endpoints with One

```
Before (SSE):
  POST /message  → Client sends requests
  GET  /sse      → Client receives responses

After (Streamable HTTP):
  POST /mcp      → Client sends requests, receives responses
```

### 2. Update Server Code

**Before (SSE):**
```typescript
import { SSEServerTransport } from "@modelcontextprotocol/sdk/server/sse.js";
const transport = new SSEServerTransport("/message", response);
```

**After (Streamable HTTP):**
```typescript
import { StreamableHTTPServerTransport } from "@modelcontextprotocol/sdk/server/streamablehttp.js";
const transport = new StreamableHTTPServerTransport({ endpoint: "/mcp" });
```

### 3. Update Client Configuration

```json
// Before
{ "url": "http://localhost:8080/sse" }

// After
{ "url": "http://localhost:8080/mcp" }
```

### 4. Test Thoroughly

- Verify initialization handshake works
- Test tool calls and responses
- Verify streaming notifications (progress updates)
- Check authentication flow if using OAuth

---

## Key Takeaway

**Don't start new projects with SSE.** Use Streamable HTTP for all new HTTP-based MCP servers. If you have existing SSE servers, plan a migration — the change is straightforward.
