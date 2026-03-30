# The AGENTS.md Standard & The Agentic AI Foundation

> History, governance, and why AGENTS.md became the universal standard for AI agent instructions.

---

## Timeline

| Date | Event |
|------|-------|
| **Early 2025** | OpenAI introduces AGENTS.md with Codex CLI. First tool to use a standardized agent instruction file. |
| **Mid 2025** | GitHub Copilot Coding Agent, Cursor, Google Jules, and others adopt AGENTS.md as a recognized format. |
| **Oct 2025** | 60,000+ open-source repositories include AGENTS.md files. |
| **Dec 2025** | OpenAI donates AGENTS.md to the Linux Foundation as a founding project of the **Agentic AI Foundation (AAIF)**. |
| **Early 2026** | AAIF stewards AGENTS.md alongside MCP (Model Context Protocol) and Goose. Cross-vendor adoption accelerates. |

---

## What Is the Agentic AI Foundation (AAIF)?

The AAIF is an industry-neutral, open-source foundation hosted by the **Linux Foundation**. Its mission is to create shared infrastructure and standards for the agentic AI era.

### Founding Members

| Company | Contribution |
|---------|-------------|
| **OpenAI** | AGENTS.md specification |
| **Anthropic** | MCP (Model Context Protocol) |
| **Block** | Goose (open-source AI agent) |
| **Google** | Jules / Gemini CLI integration |
| **Microsoft** | GitHub Copilot integration |
| **AWS** | Cloud agent infrastructure |

### Why It Matters

Without a shared standard, every AI tool would need its own instruction file:
- `.cursorrules` for Cursor
- `CLAUDE.md` for Claude Code
- `copilot-instructions.md` for Copilot
- `.gemini.md` for Gemini
- Custom formats for every new tool

AGENTS.md solves this: **write once, use everywhere.**

---

## Design Principles

The AGENTS.md specification follows these principles:

### 1. Simplicity

Plain Markdown. No custom syntax, no YAML requirements, no schema validation. Any developer can write it.

### 2. Portability

Works across all tools without modification. The same file serves Copilot, Codex, Claude, Cursor, and future tools.

### 3. Proximity

Instructions live in the repository, next to the code they describe. The nearest AGENTS.md in the directory tree takes precedence.

### 4. Progressive Disclosure

A minimal file works. You can start with 5 lines and add detail as needed. No required sections or fields.

### 5. Human-Readable

Unlike JSON/YAML config files, AGENTS.md is Markdown that humans can read, write, and review in PRs.

---

## AGENTS.md in the Broader Ecosystem

```
┌─────────────────────────────────────────────┐
│           Agentic AI Foundation              │
│                 (AAIF)                        │
├────────────┬────────────┬───────────────────┤
│ AGENTS.md  │    MCP     │     Goose          │
│            │            │                    │
│ Agent      │ Model      │ Open-source        │
│ instruction│ Context    │ AI coding          │
│ standard   │ Protocol   │ agent              │
└────────────┴────────────┴───────────────────┘
```

### How These Relate

| Standard | Purpose | Analogy |
|----------|---------|---------|
| **AGENTS.md** | What the agent should know about the project | Job description |
| **MCP** | How the agent connects to tools and data sources | API protocol |
| **Goose** | An actual agent that reads AGENTS.md and uses MCP | The employee |

---

## Adoption Numbers (March 2026)

- **60,000+** open-source repos with AGENTS.md
- **10+** AI coding tools support the format
- **6** major tech companies co-govern the standard
- **Growing** at ~5,000 new repos per month

---

## The Case for AGENTS.md

### For Individual Developers

- **One file** instead of maintaining `.cursorrules` + `CLAUDE.md` + `copilot-instructions.md`
- **Future-proof**: new tools will adopt the standard
- **Portable**: switch AI tools without rewriting instructions

### For Teams

- **Consistency**: every developer's AI assistant follows the same rules
- **Onboarding**: new AI tools work immediately with existing instructions
- **Cross-tool**: team members can use their preferred AI tool

### For Open Source

- **Contributor-friendly**: any AI tool a contributor uses will understand the project
- **Low barrier**: simple Markdown format
- **Community standard**: backed by the Linux Foundation

---

## Specification Status

AGENTS.md is currently a **de facto standard** with a simple specification:

- **Format**: Markdown (`.md`)
- **Filename**: `AGENTS.md` (case-sensitive)
- **Location**: Repository root, or any directory for scoped rules
- **Required sections**: None (progressive disclosure)
- **Recommended sections**: Setup, Conventions, Boundaries, Testing
- **Resolution**: Nearest file in directory tree wins
- **Encoding**: UTF-8

The AAIF may formalize a versioned specification in the future, but the current convention is intentionally minimal and flexible.
