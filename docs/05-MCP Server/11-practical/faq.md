# MCP Frequently Asked Questions

> Answers to the 20 most common questions about MCP.

---

## General

### 1. What is MCP in simple terms?
MCP (Model Context Protocol) is a standard that lets AI assistants connect to external tools and data sources. Think of it as USB-C for AI — one standard protocol that works with any tool and any AI client.

### 2. Who created MCP?
Anthropic created and open-sourced MCP in November 2024. It is now governed under the Linux Foundation as an open standard.

### 3. Is MCP free to use?
Yes. MCP is an open-source protocol with MIT-licensed SDKs. Anyone can build servers and clients.

### 4. What's the difference between MCP and function calling?
Function calling is a model-specific feature where tools are defined in the API request. MCP is a protocol that manages tool discovery, lifecycle, and execution across any AI client. MCP is the transport layer; function calling is how the model decides to use a tool.

### 5. Do I need to build my own MCP server?
Usually no. There are 10,000+ community servers available. Build your own only when you need to connect to proprietary systems.

---

## Setup & Configuration

### 6. Where do I configure MCP servers?
- **VS Code / Copilot**: `.vscode/mcp.json` (project) or `settings.json` (user)
- **Claude Desktop**: `claude_desktop_config.json`
- **Cursor**: `~/.cursor/mcp.json`

### 7. Can I use MCP with GitHub Copilot?
Yes. GitHub Copilot fully supports MCP servers via `.vscode/mcp.json` configuration.

### 8. Do MCP servers work across different AI clients?
Yes — that's the whole point. A server built for Claude works with Copilot, Cursor, Cline, and any other MCP-compatible client.

### 9. Can I connect multiple MCP servers at once?
Yes. Most clients support connecting to multiple servers simultaneously. The AI can use tools from all connected servers.

### 10. Do I need to install anything globally?
Most servers use `npx -y` (auto-download and run), so no global installation is needed. Python servers may require `pip install`.

---

## Development

### 11. Which language should I use to build an MCP server?
- **Python**: Use FastMCP — decorator-based, fastest to prototype
- **TypeScript**: Use the official SDK — strong typing, good for production
- Both are first-class options with official SDK support.

### 12. How many tools should a server have?
10-15 maximum. More than that and AI agents struggle to select the right one. Split into multiple focused servers if needed.

### 13. Can a tool modify files or databases?
Yes, but destructive operations should require user consent. The host mediates approval.

### 14. How do I test my MCP server?
Use MCP Inspector (`npx @modelcontextprotocol/inspector`) — it provides a web UI for testing tools, resources, and prompts interactively.

### 15. Why can't the AI see my tools?
Check: (1) Config file exists and has valid JSON, (2) Server command path is correct, (3) You restarted the client after config changes, (4) Server process isn't crashing on startup.

---

## Security

For a full treatment of trust boundaries, host mediation, and the consent framework, see [Security Model](../08-security/security-model.md).

### 16. Is MCP secure?
MCP has a layered security model with host-mediated access, user consent, and server isolation. Security depends on implementation — follow best practices (input validation, least privilege, TLS for remote). See [Security Model](../08-security/security-model.md) for details on trust layers and host responsibilities.

### 17. Can an MCP server access my private code?
Only if you configure it to. Filesystem servers are scoped to directories you specify. The `roots` feature explicitly declares which paths a server can access.

### 18. Should I worry about prompt injection?
Yes. Tool responses can contain content that tries to manipulate the AI. The host treats tool responses as untrusted data, and user consent for destructive operations is a key defense.

---

## Architecture

### 19. What's the difference between stdio and HTTP transport?
- **stdio**: Local only. Server runs as a subprocess. No network involved. Best for local tools.
- **HTTP**: Can be local or remote. Server runs as a web service. Required for cloud/shared servers.

### 20. Can MCP servers talk to each other?
No. Servers are isolated by design. They cannot see or communicate with other servers. Only the host can aggregate data across servers.
