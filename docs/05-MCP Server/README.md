# 05 — Model Context Protocol (MCP) Server

> The complete educational guide to MCP — the universal "USB-C for AI agents" that connects language models to tools, data, and services.

---

## Why This Section Exists

Without MCP, every AI agent is limited to text-in/text-out. MCP transforms agents into **tool-using systems** that can query databases, manage repositories, browse the web, run tests, and orchestrate entire workflows — all through a single, standardized protocol.

If you're building a framework for AI-assisted development, **MCP is the integration layer** that makes everything else work.

---

## Table of Contents

### 01 — Fundamentals
- [What Is MCP?](01-fundamentals/what-is-mcp.md) — The problem MCP solves and why it matters
- [History & Adoption](01-fundamentals/history-and-adoption.md) — From Anthropic's creation to industry standard
- [MCP vs Alternatives](01-fundamentals/mcp-vs-alternatives.md) — MCP vs function calling, plugin systems, and custom APIs
- [Key Terminology](01-fundamentals/terminology.md) — Glossary of MCP concepts

### 02 — Architecture
- [The Client-Host-Server Model](02-architecture/client-host-server.md) — How the three layers interact
- [Protocol Lifecycle](02-architecture/protocol-lifecycle.md) — Initialization, capability negotiation, and session management
- [Message Format & JSON-RPC](02-architecture/message-format.md) — Request/response/notification patterns

### 03 — Primitives (Core Concepts)
- [Tools](03-primitives/tools.md) — Executable functions AI agents can call
- [Resources](03-primitives/resources.md) — Read-only data exposed to models
- [Prompts](03-primitives/prompts.md) — Reusable interaction templates
- [Sampling](03-primitives/sampling.md) — Server-initiated LLM requests
- [Roots & Elicitation](03-primitives/roots-and-elicitation.md) — Workspace access and structured user input

### 04 — Transport Layer
- [Transport Overview](04-transport/transport-overview.md) — How messages travel between client and server
- [STDIO Transport](04-transport/stdio.md) — Local subprocess communication
- [Streamable HTTP](04-transport/streamable-http.md) — The modern remote transport
- [SSE (Deprecated)](04-transport/sse-deprecated.md) — Why SSE was replaced and migration guide

### 05 — Building MCP Servers
- [Your First Server (Python)](05-building-servers/first-server-python.md) — Step-by-step Python MCP server
- [Your First Server (TypeScript)](05-building-servers/first-server-typescript.md) — Step-by-step TypeScript MCP server
- [Tool Design Patterns](05-building-servers/tool-design-patterns.md) — Designing tools AI agents actually use well
- [Resource & Prompt Design](05-building-servers/resource-prompt-design.md) — Exposing data and templates effectively
- [Testing & Debugging](05-building-servers/testing-and-debugging.md) — Validating your MCP server works correctly

### 06 — Best Practices
- [Server Design Principles](06-best-practices/server-design-principles.md) — Core rules for building great MCP servers
- [Tool Naming & Schemas](06-best-practices/tool-naming-and-schemas.md) — How to name and structure tools for LLM success
- [Error Handling](06-best-practices/error-handling.md) — Actionable errors that agents can recover from
- [Performance & Context Budget](06-best-practices/performance-and-context.md) — Managing tokens, latency, and payload size

### 07 — Anti-Patterns
- [Common Mistakes](07-anti-patterns/common-mistakes.md) — Top 10 MCP server pitfalls
- [The API Wrapper Trap](07-anti-patterns/api-wrapper-trap.md) — Why 1:1 API mapping fails
- [Over-Engineering Servers](07-anti-patterns/over-engineering.md) — When servers try to be too smart

### 08 — Security
- [Security Model](08-security/security-model.md) — Trust boundaries, host mediation, and consent
- [Authentication & OAuth 2.1](08-security/authentication.md) — Token-based auth for remote MCP servers
- [Threat Landscape](08-security/threat-landscape.md) — Prompt injection, tool poisoning, and confused deputy
- [Security Checklist](08-security/security-checklist.md) — Production-ready security audit checklist

### 09 — Ecosystem
- [Popular MCP Servers](09-ecosystem/popular-servers.md) — The essential servers every developer should know
- [Client Compatibility](09-ecosystem/client-compatibility.md) — Claude, Copilot, Cursor, VS Code, and more
- [Community & Registries](09-ecosystem/community-and-registries.md) — Where to find and share MCP servers
- [MCP in Prompt Files](09-ecosystem/mcp-in-prompt-files.md) — Using MCP tools inside `.prompt.md` files

### 10 — Advanced Topics
- [Enterprise Deployment](10-advanced/enterprise-deployment.md) — Gateways, proxies, and production scaling
- [Multi-Server Orchestration](10-advanced/multi-server-orchestration.md) — Connecting multiple servers in agent workflows
- [Custom Transport](10-advanced/custom-transport.md) — Building your own transport layer
- [MCP + Meta-Prompting](10-advanced/mcp-plus-meta-prompting.md) — Integrating MCP into your instruction framework

### 11 — Practical Resources
- [Templates & Starter Code](11-practical/templates.md) — Copy-paste server templates for Python and TypeScript
- [Troubleshooting Guide](11-practical/troubleshooting.md) — When things break — diagnosis and fixes
- [FAQ](11-practical/faq.md) — 20 frequently asked questions about MCP
- [Resources & Further Reading](11-practical/resources.md) — Links, specs, courses, and community

---

## Quick Start Paths

| Goal | Start Here |
|------|-----------|
| **"What is MCP?"** | [What Is MCP?](01-fundamentals/what-is-mcp.md) |
| **Build my first server** | [First Server (Python)](05-building-servers/first-server-python.md) or [TypeScript](05-building-servers/first-server-typescript.md) |
| **Secure my deployment** | [Security Model](08-security/security-model.md) → [Checklist](08-security/security-checklist.md) |
| **Find servers to install** | [Popular MCP Servers](09-ecosystem/popular-servers.md) |
| **Integrate with my framework** | [MCP + Meta-Prompting](10-advanced/mcp-plus-meta-prompting.md) |

---

## Conventions

| Icon | Meaning |
|------|---------|
| 💡 | Tip or recommendation |
| ⚠️ | Warning or security concern |
| 📋 | Template or code example |
| 🔗 | Cross-reference to another section |

---

*Last updated: March 2026*
