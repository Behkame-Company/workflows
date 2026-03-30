# Roots & Elicitation

> Additional MCP primitives for workspace access and structured user input.

---

## Roots

### What Are Roots?

Roots define **which file system locations** the client makes available to servers. They set the boundaries of what a server can access.

```json
// Client declares its roots
{
  "roots": [
    {
      "uri": "file:///home/user/projects/webapp",
      "name": "Web Application"
    },
    {
      "uri": "file:///home/user/projects/shared-lib",
      "name": "Shared Library"
    }
  ]
}
```

### Why Roots Matter

- **Security**: Servers only see approved directories, not your entire filesystem
- **Context**: Servers understand the project structure and boundaries
- **Multi-project**: Multiple roots can represent a monorepo or related projects

### Root Operations

```json
// Server → Client: request roots
{ "method": "roots/list", "id": 1 }

// Client → Server: return available roots
{
  "result": {
    "roots": [
      { "uri": "file:///workspace/frontend", "name": "Frontend App" },
      { "uri": "file:///workspace/backend", "name": "Backend API" }
    ]
  }
}
```

Roots can change dynamically. When they do, the client sends:
```json
{ "method": "notifications/roots/list_changed" }
```

---

## Elicitation

### What Is Elicitation?

Elicitation allows MCP servers to **request structured input from the user** during a workflow. It's like showing a form or dialog.

Instead of the AI guessing what the user wants, the server can ask directly:

```json
// Server → Client: request user input
{
  "method": "elicitation/create",
  "params": {
    "message": "Which environment should I deploy to?",
    "requestedSchema": {
      "type": "object",
      "properties": {
        "environment": {
          "type": "string",
          "enum": ["staging", "production"],
          "description": "Target deployment environment"
        },
        "confirm": {
          "type": "boolean",
          "description": "Confirm you want to proceed with deployment"
        }
      },
      "required": ["environment", "confirm"]
    }
  }
}

// Client → Server: user's response
{
  "result": {
    "action": "accept",
    "content": {
      "environment": "staging",
      "confirm": true
    }
  }
}
```

### Elicitation Actions

| Action | Meaning |
|--------|---------|
| `accept` | User provided the requested input |
| `decline` | User chose not to provide input |
| `cancel` | User cancelled the entire operation |

### When to Use Elicitation

| Scenario | Why? |
|----------|------|
| **Deployment target** | Critical choice that shouldn't be AI-guessed |
| **Database migration** | User must confirm destructive operations |
| **API key input** | Sensitive data the AI shouldn't handle |
| **Ambiguous parameters** | Multiple valid options, user should choose |

---

## Roots + Elicitation in Practice

```
Workflow: Deploy updated API

1. Server reads roots to find project directories
2. Server discovers deployment config in root
3. Server elicits: "Deploy to staging or production?"
4. User selects: "staging"
5. Server uses tool to run deployment
6. Server reports results
```

---

## Key Takeaways

1. **Roots** = filesystem boundaries the client exposes to servers (security)
2. **Elicitation** = servers requesting structured user input (clarity)
3. Both are **newer primitives** — not all clients/servers support them yet
4. Roots prevent servers from accessing **unauthorized directories**
5. Elicitation prevents AI from **guessing** when user choice is critical
