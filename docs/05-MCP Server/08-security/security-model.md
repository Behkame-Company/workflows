# MCP Security Model

> Understanding trust boundaries, host mediation, and the consent framework that keeps MCP safe.

---

## The Three Trust Layers

```
┌─────────────────────────────────────────────┐
│  HOST (Claude Desktop, VS Code, IDE)        │  ← MOST TRUSTED
│  - Controls which servers can connect        │  - Makes final decisions
│  - Manages user consent                      │  - Enforces security policy
│  ┌────────────────────────────────────────┐  │
│  │  CLIENT (Protocol Handler)             │  │  ← SEMI-TRUSTED
│  │  - Translates protocol messages         │  │  - Isolated per server
│  │  - Enforces capability boundaries       │  │
│  │  ┌──────────────────────────────────┐  │  │
│  │  │  SERVER (MCP Server)             │  │  │  ← LEAST TRUSTED
│  │  │  - Provides tools/resources       │  │  │  - Untrusted code
│  │  │  - Requests permissions           │  │  │  - Cannot access other
│  │  │  - Cannot see other servers       │  │  │    servers' data
│  │  └──────────────────────────────────┘  │  │
│  └────────────────────────────────────────┘  │
└─────────────────────────────────────────────┘
```

---

## Core Security Principles

### 1. Host-Mediated Access

The host application always sits between the user, the AI, and MCP servers:

- **Tool invocation**: AI requests → Host shows user → User approves → Host executes
- **Resource access**: Server offers data → Host controls what AI sees
- **Sampling requests**: Server asks AI to think → Host mediates the request

### 2. User Consent

No destructive action should proceed without user awareness:

```
AI: "I need to delete the staging database to proceed."
Host: [CONFIRMATION DIALOG]
  "The database-server wants to execute: DROP DATABASE staging"
  [Allow] [Deny] [Allow Always for this Session]
```

### 3. Least Privilege

Each server should only have access to what it needs:

```json
{
  "mcpServers": {
    "github": {
      "permissions": {
        "repos": ["myorg/webapp"],
        "scopes": ["issues:read", "issues:write", "pull_requests:read"]
      }
    }
  }
}
```

### 4. Server Isolation

Servers cannot see each other's data or communicate directly:

```
github-server ─────── client-1 ──┐
                                  ├── HOST (aggregates)
postgres-server ────── client-2 ──┘
                 ╳
              (no direct communication)
```

---

## What Servers Should NEVER Do

| Action | Why It's Dangerous |
|--------|--------------------|
| Access other servers' data | Breaks isolation model |
| Store user credentials | Creates attack surface |
| Execute arbitrary code | Sandbox escape risk |
| Modify their own config | Privilege escalation |
| Bypass consent flows | Violates user trust |
| Log sensitive data | Information leakage |

---

## What Hosts MUST Do

| Responsibility | Implementation |
|---------------|----------------|
| Validate server identity | Check command paths, verify packages |
| Control tool approval | Show user what the tool will do before executing |
| Limit resource exposure | Filter what the AI sees from server resources |
| Monitor server behavior | Log all tool invocations for audit |
| Enforce rate limits | Prevent runaway tool invocations |
| Manage server lifecycle | Clean shutdown, restart on failure |

---

## Key Takeaways

1. **Trust flows inward**: Host > Client > Server (servers are least trusted)
2. **Consent is mandatory**: Users must approve destructive operations
3. **Servers are isolated**: They can't see or talk to each other
4. **Least privilege applies**: Grant minimum necessary access
5. **The host is the security boundary**: It enforces all policies

---

## Next Steps

- 🔗 [Authentication & OAuth 2.1](authentication.md) — How remote auth works
- 🔗 [Threat Landscape](threat-landscape.md) — Known attack vectors
