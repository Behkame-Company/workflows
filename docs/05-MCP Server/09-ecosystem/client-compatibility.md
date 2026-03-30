# Client Compatibility Matrix

> Which AI clients support MCP, what features they support, and configuration specifics.

---

## Compatibility Matrix (2025-2026)

| Client | MCP Support | Transport | Multi-Server | Sampling | Roots | Config Location |
|--------|-------------|-----------|-------------|----------|-------|----------------|
| **Claude Desktop** | ✅ Full | stdio, HTTP | ✅ | ✅ | ✅ | `claude_desktop_config.json` |
| **Claude Code** | ✅ Full | stdio, HTTP | ✅ | ✅ | ✅ | Project settings |
| **GitHub Copilot** | ✅ Full | stdio, HTTP | ✅ | ⚠️ Partial | ✅ | `.vscode/mcp.json` or `settings.json` |
| **VS Code** | ✅ Full | stdio, HTTP | ✅ | ⚠️ Partial | ✅ | `.vscode/mcp.json` |
| **Cursor** | ✅ Full | stdio, HTTP | ✅ | ⚠️ Partial | ⚠️ | Cursor settings |
| **Windsurf** | ✅ Full | stdio, HTTP | ✅ | ❌ | ❌ | `~/.codeium/windsurf/mcp_config.json` |
| **Cline** | ✅ Full | stdio, HTTP | ✅ | ✅ | ✅ | Cline settings |
| **Continue** | ✅ Full | stdio, HTTP | ✅ | ⚠️ Partial | ⚠️ | `config.json` |
| **ChatGPT** | ✅ Recent | HTTP | ✅ | ❌ | ❌ | ChatGPT settings |
| **Zed** | ✅ | stdio | ✅ | ❌ | ❌ | `settings.json` |

---

## Configuration by Client

### VS Code / GitHub Copilot
```jsonc
// .vscode/mcp.json (project-level, recommended)
{
  "servers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${env:GITHUB_TOKEN}"
      }
    }
  }
}
```

### Claude Desktop
```jsonc
// ~/Library/Application Support/Claude/claude_desktop_config.json (macOS)
// %APPDATA%\Claude\claude_desktop_config.json (Windows)
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "ghp_..."
      }
    }
  }
}
```

### Cursor
```jsonc
// ~/.cursor/mcp.json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "ghp_..."
      }
    }
  }
}
```

---

## Feature Support Details

### Primitives Support

| Primitive | Claude | Copilot | Cursor | Cline | Windsurf |
|-----------|--------|---------|--------|-------|----------|
| **Tools** | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Resources** | ✅ | ✅ | ⚠️ | ✅ | ⚠️ |
| **Prompts** | ✅ | ✅ | ⚠️ | ✅ | ❌ |
| **Sampling** | ✅ | ⚠️ | ⚠️ | ✅ | ❌ |
| **Roots** | ✅ | ✅ | ⚠️ | ✅ | ❌ |
| **Elicitation** | ✅ | ⚠️ | ❌ | ⚠️ | ❌ |

### Transport Support

| Transport | Claude | Copilot | Cursor | Cline | Windsurf |
|-----------|--------|---------|--------|-------|----------|
| **stdio** | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Streamable HTTP** | ✅ | ✅ | ✅ | ✅ | ✅ |
| **SSE (deprecated)** | ⚠️ | ⚠️ | ⚠️ | ⚠️ | ⚠️ |

---

## Key Takeaways

1. **All major AI editors support MCP** — It's the de facto standard
2. **Tools are universally supported** — The core primitive works everywhere
3. **stdio is the safest transport** — Works in every client
4. **Config locations vary** — Check your client's documentation
5. **Test across clients** — If your team uses multiple editors, verify compatibility
