# Sampling — Server-Initiated AI Requests

> Sampling flips the normal flow: the MCP server asks the AI to reason, not the other way around.

---

## What Is Sampling?

In normal MCP flow, the AI calls the server (tool invocation). **Sampling reverses this**: the server sends a prompt to the AI and gets back a completion.

```
Normal:   AI → "call this tool" → Server → result → AI
Sampling: Server → "think about this" → AI → reasoning → Server
```

This is powerful for scenarios where the server needs intelligent analysis but doesn't have its own AI model.

---

## How Sampling Works

```
1. Server sends sampling/createMessage to Client
2. Client validates the request (security, permissions)
3. Client may present to user for approval
4. Client forwards to LLM for completion
5. LLM generates response
6. Client returns result to Server
7. Server uses the AI's reasoning in its workflow
```

### Example Request
```json
{
  "method": "sampling/createMessage",
  "id": 1,
  "params": {
    "messages": [
      {
        "role": "user",
        "content": {
          "type": "text",
          "text": "Analyze these error logs and identify the root cause:\n\n2026-03-26 10:15:03 ERROR ConnectionPool exhausted\n2026-03-26 10:15:04 ERROR Query timeout after 30s\n2026-03-26 10:15:05 ERROR 503 Service Unavailable"
        }
      }
    ],
    "modelPreferences": {
      "hints": [{ "name": "claude-sonnet" }],
      "intelligencePriority": 0.8,
      "speedPriority": 0.2
    },
    "systemPrompt": "You are a DevOps expert. Analyze the logs and provide a root cause analysis with recommended fixes.",
    "maxTokens": 500
  }
}
```

### Example Response
```json
{
  "id": 1,
  "result": {
    "role": "assistant",
    "content": {
      "type": "text",
      "text": "Root cause: Database connection pool exhaustion causing cascading failures. The pool is configured for fewer connections than concurrent requests. Recommended fixes: 1) Increase pool_size to 20. 2) Add connection timeout of 5s. 3) Implement circuit breaker pattern."
    },
    "model": "claude-sonnet-4-20250514",
    "stopReason": "endTurn"
  }
}
```

---

## Sampling Security

Sampling is powerful but potentially dangerous. The MCP specification enforces strict controls:

### Human-in-the-Loop

The host **must** allow users to:
1. **See** the sampling request before it's sent to the AI
2. **Modify** the prompt if needed
3. **Approve or reject** the request
4. **Review** the AI's response before it's sent back to the server

### Trust Boundaries

- Servers **never** see raw model access — everything goes through the host
- The host can **filter, modify, or reject** sampling requests
- Model selection is a **preference**, not a demand — the host decides which model to use
- Token limits are **enforced by the host**, not the server

---

## Use Cases

| Scenario | Why Sampling? |
|----------|--------------|
| **Error analysis** | Server has logs, needs AI to interpret patterns |
| **PR title suggestions** | Server has diff, needs AI to summarize |
| **Auto-categorization** | Server has data, needs AI to classify |
| **Content generation** | Server has template + data, needs AI to write |
| **Risk assessment** | Server has metrics, needs AI to evaluate |

---

## Sampling vs Tool Calls

| Aspect | Tool Call | Sampling |
|--------|----------|----------|
| **Direction** | AI → Server | Server → AI |
| **Who reasons** | AI chooses the tool | AI provides reasoning |
| **Who acts** | Server executes | Server uses AI's answer |
| **Consent** | Host approves tool call | Host approves sampling request |

---

## Key Takeaways

1. Sampling lets servers **leverage AI reasoning** without hosting their own model
2. All sampling goes through the **host**, which enforces security and consent
3. Model selection is a **preference** — the host decides
4. Use sampling when the server has data but needs **intelligent analysis**
5. Always implement **human approval** for sampling in production

---

## Next Steps

- 🔗 [Roots & Elicitation](roots-and-elicitation.md) — Other advanced primitives
- 🔗 [Security Model](../08-security/security-model.md) — How MCP protects against abuse
