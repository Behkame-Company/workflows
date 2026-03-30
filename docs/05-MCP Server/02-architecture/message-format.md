# Message Format & JSON-RPC 2.0

> Every MCP message follows the JSON-RPC 2.0 specification. Understanding the wire format is essential for building and debugging MCP integrations.

---

## Why JSON-RPC?

MCP chose JSON-RPC 2.0 because it provides:

- **Simplicity** — Small, well-defined message format
- **Bidirectional** — Both sides can send requests
- **ID-based correlation** — Requests and responses are linked by ID
- **Notifications** — One-way messages for events and streaming
- **Language-agnostic** — JSON works everywhere

---

## The Three Message Types

### 1. Request

A message that **expects a response**. Contains a method, parameters, and a unique ID.

```json
{
  "jsonrpc": "2.0",
  "id": 42,
  "method": "tools/call",
  "params": {
    "name": "search_code",
    "arguments": {
      "query": "authentication middleware",
      "repo": "myorg/webapp"
    }
  }
}
```

| Field | Required | Description |
|-------|----------|-------------|
| `jsonrpc` | ✅ | Always `"2.0"` |
| `id` | ✅ | Unique identifier to correlate with response |
| `method` | ✅ | The RPC method name (e.g., `tools/call`, `resources/read`) |
| `params` | ⚠️ | Method parameters (optional for some methods) |

### 2. Response

A reply to a request. Contains **either** a result **or** an error.

**Success:**
```json
{
  "jsonrpc": "2.0",
  "id": 42,
  "result": {
    "content": [{
      "type": "text",
      "text": "Found 3 matching files..."
    }]
  }
}
```

**Error:**
```json
{
  "jsonrpc": "2.0",
  "id": 42,
  "error": {
    "code": -32602,
    "message": "Repository not found: myorg/webapp",
    "data": {
      "suggestion": "Check that the repository exists and you have access"
    }
  }
}
```

| Field | Required | Description |
|-------|----------|-------------|
| `jsonrpc` | ✅ | Always `"2.0"` |
| `id` | ✅ | Must match the request's ID |
| `result` | ⚠️ | Success payload (mutually exclusive with `error`) |
| `error` | ⚠️ | Error object with code, message, and optional data |

### 3. Notification

A **one-way message** that does not expect a response. No `id` field.

```json
{
  "jsonrpc": "2.0",
  "method": "notifications/progress",
  "params": {
    "progressToken": "import-task",
    "progress": 75,
    "total": 100
  }
}
```

| Field | Required | Description |
|-------|----------|-------------|
| `jsonrpc` | ✅ | Always `"2.0"` |
| `method` | ✅ | The notification method name |
| `params` | ⚠️ | Optional parameters |
| `id` | ❌ | Must NOT be present (that's what makes it a notification) |

---

## Standard Error Codes

| Code | Name | When |
|------|------|------|
| `-32700` | Parse error | Invalid JSON |
| `-32600` | Invalid request | Missing required fields |
| `-32601` | Method not found | Unknown method name |
| `-32602` | Invalid params | Wrong parameter types or missing required params |
| `-32603` | Internal error | Server-side failure |

MCP servers should also use **custom error codes** for domain-specific errors.

---

## Common MCP Methods

### Client → Server (Requests)

| Method | Purpose |
|--------|---------|
| `initialize` | Start a session, exchange capabilities |
| `tools/list` | Discover available tools |
| `tools/call` | Execute a tool |
| `resources/list` | Discover available resources |
| `resources/read` | Read a specific resource |
| `resources/subscribe` | Subscribe to resource changes |
| `prompts/list` | Discover available prompts |
| `prompts/get` | Get a specific prompt template |
| `ping` | Health check |

### Server → Client (Requests)

| Method | Purpose |
|--------|---------|
| `sampling/createMessage` | Request AI reasoning |
| `elicitation/create` | Request structured user input |
| `roots/list` | Get accessible workspace roots |

### Notifications (Either Direction)

| Method | Direction | Purpose |
|--------|-----------|---------|
| `notifications/initialized` | Client → Server | Session ready |
| `notifications/progress` | Server → Client | Task progress update |
| `notifications/tools/list_changed` | Server → Client | Tool list updated |
| `notifications/resources/list_changed` | Server → Client | Resource list updated |
| `notifications/cancelled` | Either | Cancel a pending request |

---

## Message Rules

1. All messages must be **valid JSON** and **UTF-8 encoded**
2. Over stdio, messages are **newline-delimited** (one JSON object per line)
3. Over HTTP, messages use standard HTTP request/response semantics
4. The `jsonrpc` field must always be `"2.0"`
5. Request IDs must be **unique within a session**
6. Notifications must **not** include an `id` field
7. Responses must include **either** `result` or `error`, never both

---

## Key Takeaways

1. **Three message types**: Request (expects response), Response (answers request), Notification (fire-and-forget)
2. **ID correlation**: Requests and responses are matched by their `id` field
3. **Bidirectional**: Both client and server can send requests
4. **Standard errors**: Use JSON-RPC error codes plus domain-specific codes
5. **Transport-agnostic**: Same message format works over stdio, HTTP, or custom transports
