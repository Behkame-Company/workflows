# History & Adoption of MCP

> From Anthropic's internal project to the industry standard for AI-tool integration.

---

## Timeline

### 2024 — Origin

| Date | Event |
|------|-------|
| **Late 2024** | Anthropic creates MCP as an internal protocol for Claude's tool-use capabilities |
| **November 2024** | MCP is open-sourced with initial specification and Python/TypeScript SDKs |
| **December 2024** | Early adopters (Cursor, Replit) begin implementing MCP clients |

### 2025 — Rapid Adoption

| Date | Event |
|------|-------|
| **Q1 2025** | Claude Desktop ships with native MCP client support |
| **March 2025** | SSE transport deprecated in favor of Streamable HTTP |
| **Q2 2025** | Microsoft announces MCP support in VS Code and Copilot |
| **Mid 2025** | OpenAI adds MCP compatibility to ChatGPT agent features |
| **Q3 2025** | Google integrates MCP into Gemini-based tools |
| **Late 2025** | Linux Foundation accepts MCP for governance and standardization |

### 2026 — Industry Standard

| Date | Event |
|------|-------|
| **Q1 2026** | 10,000+ community MCP servers available |
| **Q1 2026** | Enterprise gateway products emerge (Kong, MCPRelay, Storm MCP) |
| **Ongoing** | Specification continues evolving (OAuth 2.1, elicitation, roots) |

---

## Why MCP Won

Before MCP, several competing approaches existed:

| Approach | Problem |
|----------|---------|
| **OpenAI Function Calling** | Vendor-locked to OpenAI; no standard server protocol |
| **ChatGPT Plugins** | Deprecated; REST-based; no bidirectional communication |
| **LangChain Tools** | Framework-specific; Python-only; high abstraction overhead |
| **Custom API wrappers** | One-off integrations; no discoverability; no standard auth |

MCP won because it offered:

1. **Vendor neutrality** — Any AI model, any client, any server
2. **Bidirectional communication** — Server can request AI reasoning (sampling)
3. **Built-in security** — Host-mediated permissions and user consent
4. **Transport flexibility** — Local (stdio) or remote (HTTP) with the same protocol
5. **Open governance** — Linux Foundation stewardship prevents vendor capture

---

## Current Adoption Map

### AI Assistants (Clients)

| Tool | MCP Support | Notes |
|------|-------------|-------|
| **Claude Desktop** | ✅ Native | First MCP client; full support |
| **Claude Code** | ✅ Native | CLI with MCP server management |
| **VS Code (Copilot)** | ✅ Native | Via settings and extensions |
| **GitHub Copilot CLI** | ✅ Native | Full tool integration |
| **Cursor** | ✅ Native | Early adopter |
| **Windsurf** | ✅ Native | MCP server configuration |
| **Cline** | ✅ Native | Community-driven MCP support |
| **Roo Code** | ✅ Native | Full spec support |
| **ChatGPT** | ✅ Partial | Agent mode with MCP bridges |

### Enterprise Platforms

| Platform | MCP Support | Use Case |
|----------|-------------|----------|
| **Kong Gateway** | ✅ | Enterprise MCP proxy/gateway |
| **MCPRelay** | ✅ | Multi-tenant MCP routing |
| **Storm MCP** | ✅ | Managed MCP infrastructure |
| **Microsoft MCP Gateway** | ✅ | Azure-integrated MCP proxy |

---

## Ecosystem Growth

```
MCP Servers Available (approximate):
2024 Q4:     ~50 servers
2025 Q1:     ~500 servers
2025 Q2:    ~2,000 servers
2025 Q4:    ~5,000 servers
2026 Q1:   ~10,000+ servers
```

### Most Popular Categories

1. **Developer Tools** — GitHub, GitLab, Jira, Linear
2. **Databases** — Postgres, Supabase, MongoDB, Redis
3. **Documentation** — Context7 (live docs), web scraping
4. **Testing** — Playwright, Selenium, test runners
5. **Design** — Figma, image processing
6. **Observability** — Sentry, Datadog, logging
7. **Cloud** — AWS, Azure, GCP management
8. **Communication** — Slack, email, notifications

---

## Specification Versions

| Version | Date | Key Changes |
|---------|------|-------------|
| **2024-11-05** | Nov 2024 | Initial release: tools, resources, prompts, stdio, SSE |
| **2025-03-26** | Mar 2025 | Streamable HTTP replaces SSE; OAuth 2.1; elicitation |
| **Ongoing** | 2026 | Roots, enhanced sampling, capability extensions |

The specification follows semantic versioning with backward compatibility guarantees for the core protocol.

---

## Governance

MCP is governed by the **Linux Foundation**, ensuring:

- No single vendor controls the standard
- Open contribution process
- Clear specification versioning
- Community-driven evolution

### Contributing

- **Specification**: [github.com/modelcontextprotocol/specification](https://github.com/modelcontextprotocol/specification)
- **SDKs**: Official Python and TypeScript SDKs maintained by Anthropic
- **Community servers**: Open-source ecosystem on GitHub

---

## Key Takeaways

1. MCP went from internal Anthropic project to industry standard in ~18 months
2. Every major AI coding tool now supports MCP
3. The ecosystem has grown to 10,000+ servers
4. Linux Foundation governance ensures vendor neutrality
5. The specification continues evolving with real-world needs

---

## Next Steps

- 🔗 [MCP vs Alternatives](mcp-vs-alternatives.md) — Detailed comparison with other approaches
- 🔗 [Client Compatibility](../09-ecosystem/client-compatibility.md) — Which tools support what

---

*"The fastest-adopted open protocol in AI history."*
