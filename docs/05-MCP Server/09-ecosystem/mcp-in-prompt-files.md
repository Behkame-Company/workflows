# MCP Tools in Prompt Files

> How to reference and use MCP server tools inside `.prompt.md` reusable prompt files.

---

## The `tools:` Field

Prompt files (`.prompt.md`) can declare which tools they need using YAML frontmatter:

```markdown
---
mode: agent
tools:
  - githubRepo
  - codebase
  - context7-mcp
description: "Review code changes and suggest improvements"
---

Review the recent changes in this repository and suggest improvements
for code quality, performance, and security.
```

---

## Tool Types Available

### 1. Built-in VS Code Tools
```yaml
tools:
  - codebase          # Search across workspace files
  - githubRepo        # Interact with the connected GitHub repo
  - terminalLastCommand  # Access last terminal output
  - editFiles         # Propose file edits
```

### 2. Extension-Provided Tools
```yaml
tools:
  - copilot_codeReview  # Copilot code review
```

### 3. MCP Server Tools
Reference tools from your configured MCP servers:
```yaml
tools:
  - context7-mcp        # Library documentation lookup
  - github-mcp-server   # Full GitHub API access
  - postgres-mcp        # Database queries
```

**Format**: Use the server name as configured in `.vscode/mcp.json` or settings.

---

## Example Prompt Files with MCP Tools

### Database-Aware Code Review
```markdown
---
mode: agent
tools:
  - codebase
  - postgres-mcp
description: "Review SQL queries against actual database schema"
---

# Database-Aware Code Review

Check the SQL queries in the current file against the actual database schema.
For each query:
1. Verify table and column names exist
2. Check for missing indexes
3. Suggest optimizations
```

### Documentation Generator
```markdown
---
mode: agent
tools:
  - codebase
  - context7-mcp
description: "Generate docs with up-to-date API references"
---

# Documentation Generator

Generate comprehensive documentation for the selected code.
Use context7 to verify that API usage examples match the current 
version of each library used in the project.
```

### Full-Stack Debug Assistant
```markdown
---
mode: agent
tools:
  - codebase
  - postgres-mcp
  - sentry-mcp
  - githubRepo
description: "Debug issues using code, data, and error tracking"
---

# Debug Assistant

I'll investigate the reported issue by:
1. Checking Sentry for related errors
2. Reviewing the relevant code
3. Querying the database for related data
4. Searching GitHub issues for prior reports
```

---

## Best Practices

| Practice | Description |
|----------|-------------|
| **Declare only what's needed** | Don't include tools the prompt won't use |
| **Use `mode: agent`** | Required for tool-using prompts |
| **Document tool purpose** | Add comments about why each tool is included |
| **Fallback gracefully** | Prompt should work even if a tool is unavailable |
| **Team consistency** | Share `.prompt.md` files via version control |

---

## Key Takeaways

1. **`tools:` in frontmatter** connects prompt files to MCP capabilities
2. **Three tool types**: Built-in, extension, and MCP server tools
3. **Use `mode: agent`** for any prompt that uses tools
4. **Declare minimally** — only tools the prompt actually needs
5. **Version control** your prompt files for team consistency

---

## Next Steps

- 🔗 [Popular Servers](popular-servers.md) — Top servers to configure
- 🔗 [Prompt Files Guide](../../01-Meta-Prompting Framework & AI-Assisted Development Guide/02-copilot-instructions/prompt-files.md) — Full prompt file documentation
