# Authentication & OAuth 2.1

> How MCP handles authentication for remote servers, token management, and the OAuth 2.1 flow.

---

## Authentication Landscape

| Transport | Auth Method | Typical Use |
|-----------|-------------|-------------|
| **stdio** (local) | None needed — inherits process permissions | Local tools, file access |
| **HTTP** (local) | Optional API keys via env vars | Local database, services |
| **HTTP** (remote) | OAuth 2.1 with PKCE | Cloud services, SaaS APIs |
| **HTTP** (enterprise) | mTLS + OAuth 2.1 | Internal services |

---

## OAuth 2.1 Flow for MCP

MCP's authorization specification is based on OAuth 2.1, which requires PKCE for all clients and deprecates the implicit grant flow. This replaces OAuth 2.0's optional PKCE and removes the implicit grant entirely.

```
┌────────┐     ┌─────────────────┐     ┌──────────┐     ┌──────────────┐
│  User  │────▶│  Host (Client)  │────▶│  MCP     │────▶│  Auth Server │
│        │     │                 │     │  Server  │     │  (e.g. GitHub)│
└────────┘     └─────────────────┘     └──────────┘     └──────────────┘
                       │                                        │
                       │  1. Discover auth requirements         │
                       │─────────────────────────────▶          │
                       │  2. Redirect user to login             │
                       │──────────────────────────────────────▶ │
                       │  3. User authenticates                 │
                       │  4. Receive authorization code         │
                       │◀────────────────────────────────────── │
                       │  5. Exchange code for tokens           │
                       │──────────────────────────────────────▶ │
                       │  6. Receive access + refresh tokens    │
                       │◀────────────────────────────────────── │
                       │  7. Include token in MCP requests      │
                       │─────────────────────────────▶          │
```

### Key OAuth 2.1 Requirements

- **PKCE is mandatory** — Prevents authorization code interception
- **No implicit grant** — Removed in OAuth 2.1
- **Refresh tokens required** — For long-running sessions
- **Dynamic client registration** — Clients can register without pre-configuration

---

## Practical Configuration

### Simple API Key (Local)
```json
{
  "mcpServers": {
    "database": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "DATABASE_URL": "${env:DATABASE_URL}"
      }
    }
  }
}
```

### OAuth 2.1 (Remote)
```json
{
  "mcpServers": {
    "github-remote": {
      "url": "https://mcp.github.com/v1",
      "auth": {
        "type": "oauth2",
        "authorization_url": "https://github.com/login/oauth/authorize",
        "token_url": "https://github.com/login/oauth/access_token",
        "scopes": ["repo", "read:org"],
        "pkce": true
      }
    }
  }
}
```

---

## Token Security Best Practices

| Practice | Why |
|----------|-----|
| **Never log tokens** | Tokens in logs = tokens leaked |
| **Use short-lived access tokens** | 1-hour expiry reduces blast radius |
| **Rotate refresh tokens** | Detect token theft via one-time use |
| **Scope narrowly** | Request minimum scopes needed |
| **Store securely** | Use OS keychain, not plain files |
| **Validate on every request** | Don't cache auth decisions |

---

## Key Takeaways

1. **Local = simple** — stdio inherits process permissions, no auth needed
2. **Remote = OAuth 2.1** — With mandatory PKCE, no exceptions
3. **Tokens are secrets** — Never log, always encrypt at rest
4. **Scope narrowly** — Request minimum necessary permissions
5. **Dynamic registration** — New clients can register automatically
