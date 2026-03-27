# MCP + Meta-Prompting Framework Integration

> How to weave MCP servers into your copilot-instructions.md, AGENTS.md, and spec-driven development workflow.

---

## The Integration Vision

MCP servers become part of your AI development framework:

```
┌─────────────────────────────────────────────────────┐
│  META-PROMPTING FRAMEWORK                           │
│                                                     │
│  copilot-instructions.md                            │
│    → Defines which MCP servers to prefer            │
│    → Sets conventions for tool usage                │
│                                                     │
│  AGENTS.md                                          │
│    → Per-directory MCP server assignments           │
│    → "This agent uses the postgres-mcp server"      │
│                                                     │
│  .prompt.md files                                   │
│    → tools: [github-mcp, postgres-mcp]              │
│    → Reusable workflows combining MCP tools         │
│                                                     │
│  Specs (requirements.md)                            │
│    → Define WHAT to build                           │
│    → MCP tools help VERIFY the build                │
│                                                     │
│  MCP Servers                                        │
│    → Provide the capabilities                       │
│    → GitHub, DB, monitoring, docs, testing          │
└─────────────────────────────────────────────────────┘
```

---

## Integration Points

### 1. copilot-instructions.md — Global MCP Guidance

```markdown
## MCP Server Usage

When working with this codebase, the following MCP servers are available:

- **github-mcp**: Use for all repository operations (issues, PRs, code search)
- **postgres-mcp**: Use for database queries. Always use parameterized queries.
- **context7-mcp**: Use to look up current documentation for any library
- **sentry-mcp**: Use to check for related production errors before fixing bugs

### Conventions
- Always check Sentry for related errors before starting a bug fix
- Use context7 to verify library APIs instead of guessing from training data
- Create GitHub issues for any discovered problems during development
```

### 2. AGENTS.md — Directory-Level MCP Rules

```markdown
# /src/database/
## Agent Instructions
- Use postgres-mcp to validate schema changes against the actual database
- Always run a SELECT query to verify table structure before writing migrations
- Test queries against staging database, never production

# /src/api/
## Agent Instructions  
- Use github-mcp to check for related issues before implementing endpoints
- Use sentry-mcp to review error patterns for existing endpoints
```

### 3. Prompt Files — Workflow Automation

```markdown
---
mode: agent
tools:
  - codebase
  - github-mcp-server
  - sentry-mcp
  - postgres-mcp
description: "Complete bug fix workflow"
---

# Bug Fix Workflow

1. **Investigate**: Check Sentry for the error's stack trace and frequency
2. **Find**: Search the codebase for the relevant code
3. **Verify**: Query the database to understand the data state
4. **Fix**: Implement the fix
5. **Test**: Verify the fix addresses the Sentry error
6. **Report**: Create a GitHub PR with findings
```

### 4. Specs — MCP-Verified Requirements

```markdown
# Feature: User Search API

## Requirements
- MUST return results in <200ms
- MUST search by name, email, or username
- MUST paginate at 20 results per page

## Verification (MCP-assisted)
- Use postgres-mcp to verify search index exists
- Use playwright-mcp to test the UI search component
- Use sentry-mcp to monitor for errors after deployment
```

---

## Recommended Server Sets by Project Type

### Web Application
```json
{
  "servers": {
    "github": {},
    "postgres": {},
    "context7": {},
    "playwright": {},
    "sentry": {}
  }
}
```

### API Backend
```json
{
  "servers": {
    "github": {},
    "postgres": {},
    "context7": {},
    "docker": {},
    "sentry": {}
  }
}
```

### Full-Stack + Design
```json
{
  "servers": {
    "github": {},
    "postgres": {},
    "context7": {},
    "playwright": {},
    "figma": {},
    "sentry": {}
  }
}
```

---

## Key Takeaways

1. **MCP servers are capabilities** — your framework orchestrates them
2. **Document in copilot-instructions.md** — which servers exist and when to use them
3. **Scope in AGENTS.md** — different directories may use different servers
4. **Automate with prompt files** — create reusable workflows combining tools
5. **Verify with specs** — use MCP tools to validate requirements are met

---

## Next Steps

- 🔗 [Templates](../11-practical/templates.md) — Ready-to-use configuration templates
- 🔗 [Building Your Framework](../../01-Meta-Prompting Framework & AI-Assisted Development Guide/05-meta-prompting-framework/building-your-framework.md) — Framework architecture
