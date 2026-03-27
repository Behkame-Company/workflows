# Protocol Lifecycle

> How MCP sessions begin, negotiate capabilities, exchange messages, and gracefully shut down.

---

## The Four Phases

```
Phase 1: Initialization
  Client ──── initialize ────→ Server
  Client ←── capabilities ──── Server
  Client ──── initialized ───→ Server (notification)

Phase 2: Discovery
  Client ──── tools/list ────→ Server
  Client ──── resources/list → Server
  Client ──── prompts/list ──→ Server

Phase 3: Operation
  Client ←──→ Server (requests, responses, notifications)
  (This is where the actual work happens)

Phase 4: Shutdown
  Client ──── close ─────────→ Server
  (Or transport disconnects)
```

---

## Phase 1: Initialization

### The Handshake

When a client connects to a server, they exchange capabilities:

```json
// Client → Server: initialize request
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "initialize",
  "params": {
    "protocolVersion": "2025-03-26",
    "capabilities": {
      "roots": { "listChanged": true },
      "sampling": {}
    },
    "clientInfo": {
      "name": "VS Code",
      "version": "1.87.0"
    }
  }
}

// Server → Client: initialize response
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "protocolVersion": "2025-03-26",
    "capabilities": {
      "tools": { "listChanged": true },
      "resources": { "subscribe": true },
      "prompts": { "listChanged": true }
    },
    "serverInfo": {
      "name": "github-mcp-server",
      "version": "2.1.0"
    }
  }
}

// Client → Server: initialized notification
{
  "jsonrpc": "2.0",
  "method": "notifications/initialized"
}
```

### What Capability Negotiation Does

- Both sides declare **what features they support**
- If the server supports `tools`, the client knows it can call `tools/list`
- If the client supports `sampling`, the server knows it can request LLM completions
- **Progressive enhancement**: new features don't break old implementations

---

## Phase 2: Discovery

After initialization, the client discovers what the server offers:

### Tool Discovery
```json
// Client → Server
{ "method": "tools/list", "id": 2 }

// Server → Client
{
  "result": {
    "tools": [
      {
        "name": "create_issue",
        "description": "Create a new GitHub issue",
        "inputSchema": {
          "type": "object",
          "properties": {
            "owner": { "type": "string" },
            "repo": { "type": "string" },
            "title": { "type": "string" },
            "body": { "type": "string" }
          },
          "required": ["owner", "repo", "title"]
        }
      }
    ]
  }
}
```

### Resource Discovery
```json
// Client → Server
{ "method": "resources/list", "id": 3 }

// Server → Client
{
  "result": {
    "resources": [
      {
        "uri": "github://repos/myorg/myrepo/readme",
        "name": "Repository README",
        "mimeType": "text/markdown"
      }
    ]
  }
}
```

### Dynamic Updates

Servers can notify clients when their capabilities change:

```json
// Server → Client (notification)
{
  "method": "notifications/tools/list_changed"
}
```

The client then re-fetches the tool list to get the updated definitions.

---

## Phase 3: Operation

This is where the actual work happens. The main message patterns:

### Tool Call (Client → Server → Client)
```json
// Client → Server: call a tool
{
  "method": "tools/call",
  "id": 10,
  "params": {
    "name": "create_issue",
    "arguments": {
      "owner": "myorg",
      "repo": "webapp",
      "title": "Fix login timeout"
    }
  }
}

// Server → Client: tool result
{
  "id": 10,
  "result": {
    "content": [{
      "type": "text",
      "text": "Created issue #247: Fix login timeout"
    }],
    "isError": false
  }
}
```

### Sampling (Server → Client → Server)
```json
// Server → Client: request AI reasoning
{
  "method": "sampling/createMessage",
  "id": 20,
  "params": {
    "messages": [{
      "role": "user",
      "content": { "type": "text", "text": "Summarize these error logs..." }
    }],
    "maxTokens": 500
  }
}

// Client → Server: AI response
{
  "id": 20,
  "result": {
    "role": "assistant",
    "content": { "type": "text", "text": "The errors indicate a database connection timeout..." }
  }
}
```

### Progress Notifications
```json
// Server → Client: progress update
{
  "method": "notifications/progress",
  "params": {
    "progressToken": "task-1",
    "progress": 50,
    "total": 100,
    "message": "Processing files..."
  }
}
```

---

## Phase 4: Shutdown

Sessions end through:

1. **Graceful close**: Client sends close message, server acknowledges
2. **Transport disconnect**: The connection drops (stdio process exits, HTTP connection closes)
3. **Timeout**: Server hasn't responded within configured limits

---

## Session State Machine

```
             ┌─── initialize ───┐
             │                   ▼
  [Created] ─┤             [Initializing]
             │                   │
             │              initialized
             │                   │
             │                   ▼
             │            [Ready / Operating]
             │                   │
             │              close / disconnect
             │                   │
             └───────────→ [Closed]
```

---

## Key Takeaways

1. **Initialization is a handshake** — both sides declare what they can do
2. **Discovery is dynamic** — tools/resources can change during a session
3. **Operations are JSON-RPC** — standard request/response/notification patterns
4. **Sampling inverts the flow** — servers can ask the AI to think
5. **Sessions are stateful** — state persists across messages within a session

---

## Next Steps

- 🔗 [Message Format & JSON-RPC](message-format.md) — The wire format in detail
- 🔗 [Tools](../03-primitives/tools.md) — The most used operational primitive
