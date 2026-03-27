# MCP Server Templates

> Copy-paste templates for building MCP servers in Python and TypeScript.

---

## Python Server Template (FastMCP)

### Minimal Server
```python
"""Minimal MCP server template."""
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("my-server")

@mcp.tool
async def hello(name: str) -> str:
    """Say hello to someone."""
    return f"Hello, {name}!"

if __name__ == "__main__":
    mcp.run()
```

### Full-Featured Server
```python
"""Full-featured MCP server with tools, resources, and prompts."""
import json
import sys
from mcp.server.fastmcp import FastMCP

mcp = FastMCP(
    "project-server",
    version="1.0.0",
    description="Project management MCP server"
)

# ─── TOOLS ──────────────────────────────────────

@mcp.tool
async def search_items(
    query: str,
    status: str = "all",
    limit: int = 20
) -> str:
    """Search project items by keyword.
    
    Args:
        query: Search terms
        status: Filter by status (all, open, closed, in_progress)
        limit: Maximum results to return (1-100, default 20)
    """
    if limit > 100:
        return "Error: Maximum limit is 100. Please reduce your limit."
    
    try:
        results = await do_search(query, status, limit)
        if not results:
            return f"No items found for '{query}' with status='{status}'."
        
        formatted = "\n".join(
            f"- [{r['id']}] {r['title']} ({r['status']})"
            for r in results
        )
        total = len(results)
        return f"Found {total} items:\n{formatted}"
    except Exception as e:
        return f"Error searching items: {str(e)}. Try simplifying your query."

@mcp.tool
async def create_item(
    title: str,
    description: str = "",
    priority: str = "medium"
) -> str:
    """Create a new project item.
    
    Args:
        title: Item title (required)
        description: Detailed description
        priority: Priority level (low, medium, high, critical)
    """
    if priority not in ("low", "medium", "high", "critical"):
        return f"Error: Invalid priority '{priority}'. Use: low, medium, high, critical."
    
    item = await do_create(title, description, priority)
    return f"Created item #{item['id']}: {title} (priority: {priority})"

# ─── RESOURCES ───────────────────────────────────

@mcp.resource("project://status")
async def project_status() -> str:
    """Current project status summary."""
    stats = await get_stats()
    return json.dumps(stats, indent=2)

@mcp.resource("project://items/{item_id}")
async def get_item(item_id: str) -> str:
    """Get details for a specific project item."""
    item = await get_item_by_id(item_id)
    return json.dumps(item, indent=2)

# ─── PROMPTS ─────────────────────────────────────

@mcp.prompt
async def weekly_review() -> str:
    """Generate a weekly project review."""
    return """Review the current project status and provide:
1. Summary of completed work this week
2. Items still in progress  
3. Blockers or risks
4. Recommended priorities for next week"""

# ─── ENTRY POINT ─────────────────────────────────

if __name__ == "__main__":
    mcp.run()
```

### Setup (pyproject.toml)
```toml
[project]
name = "project-server"
version = "1.0.0"
dependencies = ["mcp[cli]>=1.0.0"]

[project.scripts]
project-server = "project_server:mcp.run"
```

---

## TypeScript Server Template

### Minimal Server
```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";

const server = new McpServer({
  name: "my-server",
  version: "1.0.0"
});

server.tool("hello", { name: z.string() }, async ({ name }) => ({
  content: [{ type: "text", text: `Hello, ${name}!` }]
}));

const transport = new StdioServerTransport();
await server.connect(transport);
```

### Full-Featured Server
```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";

const server = new McpServer({
  name: "project-server",
  version: "1.0.0"
});

// ─── TOOLS ──────────────────────────────────────

server.tool(
  "search_items",
  "Search project items by keyword",
  {
    query: z.string().describe("Search terms"),
    status: z.enum(["all", "open", "closed"]).default("all"),
    limit: z.number().min(1).max(100).default(20)
  },
  async ({ query, status, limit }) => {
    try {
      const results = await doSearch(query, status, limit);
      const text = results.length
        ? results.map(r => `- [${r.id}] ${r.title} (${r.status})`).join("\n")
        : `No items found for '${query}'.`;
      
      return { content: [{ type: "text", text }] };
    } catch (error) {
      return {
        content: [{ type: "text", text: `Error: ${error.message}` }],
        isError: true
      };
    }
  }
);

server.tool(
  "create_item",
  "Create a new project item",
  {
    title: z.string().describe("Item title"),
    description: z.string().optional().describe("Detailed description"),
    priority: z.enum(["low", "medium", "high", "critical"]).default("medium")
  },
  async ({ title, description, priority }) => {
    const item = await doCreate(title, description ?? "", priority);
    return {
      content: [{ type: "text", text: `Created #${item.id}: ${title}` }]
    };
  }
);

// ─── RESOURCES ───────────────────────────────────

server.resource(
  "project-status",
  "project://status",
  async () => ({
    contents: [{
      uri: "project://status",
      mimeType: "application/json",
      text: JSON.stringify(await getStats(), null, 2)
    }]
  })
);

// ─── START ───────────────────────────────────────

const transport = new StdioServerTransport();
await server.connect(transport);
```

---

## Client Configuration Template

```jsonc
// .vscode/mcp.json
{
  "servers": {
    // Local Python server
    "project": {
      "command": "python",
      "args": ["-m", "project_server"],
      "env": {
        "PROJECT_DB": "${env:PROJECT_DB}"
      }
    },
    // Local TypeScript server
    "tools": {
      "command": "npx",
      "args": ["-y", "my-tools-server"],
      "env": {}
    },
    // Remote server
    "cloud-api": {
      "url": "https://mcp.example.com/v1"
    }
  }
}
```

---

## Next Steps

- 🔗 [Troubleshooting](troubleshooting.md) — When things go wrong
- 🔗 [Testing & Debugging](../05-building-servers/testing-and-debugging.md) — Test your server
