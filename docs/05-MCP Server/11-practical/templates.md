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
"""Full-featured MCP server skeleton — tools, resources, and prompts."""
import json
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("project-server", version="1.0.0")

@mcp.tool
async def search_items(query: str, status: str = "all", limit: int = 20) -> str:
    """Search project items by keyword."""
    # ... implementation

@mcp.resource("project://status")
async def project_status() -> str:
    """Current project status summary."""
    # ... implementation

@mcp.prompt
async def weekly_review() -> str:
    """Generate a weekly project review."""
    # ... implementation

if __name__ == "__main__":
    mcp.run()
```

For the complete walkthrough with error handling, validation, and testing, see [First Server (Python)](../05-building-servers/first-server-python.md).

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

const server = new McpServer({ name: "project-server", version: "1.0.0" });

server.tool("search_items", "Search project items by keyword", {
  query: z.string(), status: z.enum(["all", "open", "closed"]).default("all"),
  limit: z.number().min(1).max(100).default(20)
}, async ({ query, status, limit }) => {
  // ... implementation
});

server.resource("project-status", "project://status", async () => ({
  contents: [{ uri: "project://status", mimeType: "application/json", text: "{}" }]
}));

const transport = new StdioServerTransport();
await server.connect(transport);
```

For the complete walkthrough with error handling and testing, see [First Server (TypeScript)](../05-building-servers/first-server-typescript.md).

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
