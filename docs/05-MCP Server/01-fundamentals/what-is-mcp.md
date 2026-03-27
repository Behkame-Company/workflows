# What Is MCP (Model Context Protocol)?

> The universal standard for connecting AI agents to tools, data sources, and external services — often called "the USB-C for AI."

---

## The Problem MCP Solves

Without MCP, every AI coding tool operates in isolation:

```
Before MCP:
┌─────────┐     custom code      ┌──────────┐
│  Claude  │ ──────────────────── │  GitHub  │
│  Copilot │ ──────────────────── │  Postgres│
│  Cursor  │ ──────────────────── │  Jira    │
│  ChatGPT │ ──────────────────── │  Sentry  │
└─────────┘                      └──────────┘
   4 tools × 4 services = 16 custom integrations
```

This creates the **N×M integration problem**: every tool needs a custom connector for every service. Each connector is bespoke, brittle, and expensive to maintain.

```
With MCP:
┌─────────┐                      ┌──────────┐
│  Claude  │                      │  GitHub  │
│  Copilot │ ── MCP Protocol ──── │  Postgres│
│  Cursor  │                      │  Jira    │
│  ChatGPT │                      │  Sentry  │
└─────────┘                      └──────────┘
   All tools × All services = 1 standard protocol
```

MCP turns **N×M** into **N+M**: build one MCP server per service, one MCP client per tool, and everything connects.

---

## Definition

**Model Context Protocol (MCP)** is an open specification that defines a standard way for:

1. **AI models** (LLMs) to **discover** what tools and data are available
2. **AI models** to **invoke** tools with structured parameters
3. **AI models** to **read** data from external sources
4. **External services** to **request** AI reasoning when needed

It's the communication layer that turns a text-generation model into a **tool-using agent**.

---

## The USB-C Analogy

Just as USB-C provides one standard connector for:
- Charging
- Data transfer
- Video output
- Audio

MCP provides one standard protocol for:
- **Tool execution** (actions the AI can take)
- **Data access** (information the AI can read)
- **Prompt templates** (reusable interaction patterns)
- **Sampling** (requesting AI reasoning)

---

## What MCP Is NOT

| MCP Is | MCP Is Not |
|--------|-----------|
| A communication protocol | A replacement for your API |
| An open specification | Proprietary to any vendor |
| Tool-agnostic | Only for Claude or Copilot |
| A standard interface | An AI model itself |
| JSON-RPC based | REST or GraphQL |

---

## Who Created MCP?

MCP was created by **Anthropic** in late 2024 and released as an open specification. By 2025-2026, it has been adopted by:

- **Anthropic** (Claude Desktop, Claude Code)
- **Microsoft** (VS Code, GitHub Copilot)
- **Google** (Gemini integrations)
- **OpenAI** (ChatGPT agent mode)
- **Cursor**, **Windsurf**, **Roo Code**, **Cline**
- The **Linux Foundation** (governance and standards)

---

## Core Value Proposition

### For Development Teams

| Benefit | Description |
|---------|-------------|
| **Tool integration** | AI agents can use any MCP-compatible tool without custom code |
| **Standardization** | One protocol works across all AI assistants |
| **Security** | Built-in permission model with human-in-the-loop consent |
| **Composability** | Mix and match servers — each focused on one capability |
| **Community** | Thousands of pre-built servers for common services |

### For Your AI Framework

MCP is the missing layer between your instruction files and the real world:

```
Your Framework Stack:
┌─────────────────────────────┐
│  copilot-instructions.md    │  ← What the AI knows
│  AGENTS.md                  │  ← How the AI behaves
│  Specifications (SDD)       │  ← What to build
├─────────────────────────────┤
│  MCP Servers                │  ← What the AI can DO  ← THIS
├─────────────────────────────┤
│  Your Codebase              │  ← What gets built
└─────────────────────────────┘
```

Without MCP, your AI can only suggest text. With MCP, it can:
- Query your database schema before generating migrations
- Check Sentry for recent errors before debugging
- Read Figma designs before implementing UI
- Run tests after writing code
- Create GitHub issues from requirements

---

## A Simple Example

Here's what an MCP interaction looks like in practice:

```
Developer: "Create a GitHub issue for the login bug we found"

AI Agent (internal process):
  1. Discovers tools: [github/create_issue, github/list_repos, ...]
  2. Calls github/create_issue with:
     {
       "owner": "myorg",
       "repo": "webapp",
       "title": "Login page returns 500 on invalid email format",
       "body": "## Bug\nLogin fails with 500 error...",
       "labels": ["bug", "priority:high"]
     }
  3. Returns result to developer

AI Agent: "Created issue #247 in myorg/webapp with labels bug and priority:high."
```

The developer never leaves their editor. The AI agent used the GitHub MCP server to perform a real action.

---

## Key Takeaways

1. **MCP is a protocol, not a product** — any tool can implement it
2. **It solves the integration problem** — N+M instead of N×M
3. **It's the "action layer"** in your AI framework — without it, AI can only talk
4. **It's industry standard** — adopted by every major AI coding tool
5. **It's composable** — add/remove servers like plugins, each doing one thing well

---

## Next Steps

- 🔗 [History & Adoption](history-and-adoption.md) — How MCP became the industry standard
- 🔗 [The Client-Host-Server Model](../02-architecture/client-host-server.md) — Understand the architecture
- 🔗 [Your First Server (Python)](../05-building-servers/first-server-python.md) — Build something in 10 minutes

---

*"MCP is to AI agents what HTTP is to the web — the invisible protocol that makes everything possible."*
