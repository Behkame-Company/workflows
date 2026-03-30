# MCP Tools in Agentic Workflows

> How the Model Context Protocol extends agent capabilities beyond built-in shell and file tools — connecting agents to databases, APIs, browsers, and custom business logic.

---

## MCP in the Agentic Context

Built-in tools (shell, file read/write) handle the fundamentals. But real-world coding agents often need more: database access, API integration, browser automation, project management data. **MCP (Model Context Protocol)** provides a standardized way to expose these capabilities to agents.

```
┌──────────────────────────────────────────────────────────────────┐
│               MCP IN AGENTIC WORKFLOWS                           │
│                                                                  │
│  ┌──────────────────────────────────────────────────────┐       │
│  │                  CODING AGENT                         │       │
│  │                                                       │       │
│  │  Built-in Tools          MCP Tools                    │       │
│  │  ┌──────────────┐       ┌──────────────────────────┐ │       │
│  │  │ • Shell       │       │ • Database queries       │ │       │
│  │  │ • File read   │       │ • GitHub API             │ │       │
│  │  │ • File write  │       │ • Jira/Linear tickets    │ │       │
│  │  │ • Search      │       │ • Web search             │ │       │
│  │  │ • Git         │       │ • Browser automation     │ │       │
│  │  └──────────────┘       │ • Slack notifications    │ │       │
│  │                          │ • Custom business logic  │ │       │
│  │                          └──────────────────────────┘ │       │
│  └──────────────────────────────────────────────────────┘       │
│                                                                  │
│                         ┌──────────┐                            │
│                         │   MCP    │                            │
│                         │ Protocol │                            │
│                         └────┬─────┘                            │
│              ┌───────────────┼───────────────┐                  │
│              │               │               │                  │
│              ▼               ▼               ▼                  │
│     ┌──────────────┐ ┌──────────────┐ ┌──────────────┐         │
│     │  MCP Server  │ │  MCP Server  │ │  MCP Server  │         │
│     │  PostgreSQL   │ │  GitHub      │ │  Custom API  │         │
│     └──────────────┘ └──────────────┘ └──────────────┘         │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

## How Agents Use MCP Tools

The agent doesn't know or care that a tool is "MCP" vs "built-in." It sees a tool description and decides whether to call it based on the task.

```typescript
// From the agent's perspective, MCP tools look like any other tool
const availableTools = [
  // Built-in
  { name: "shell", description: "Execute shell commands" },
  { name: "edit_file", description: "Edit file contents" },
  
  // MCP tools (injected from connected MCP servers)
  { name: "github_create_pr", description: "Create a GitHub pull request" },
  { name: "postgres_query", description: "Run SQL query against the database" },
  { name: "linear_get_issue", description: "Fetch issue details from Linear" },
  { name: "web_search", description: "Search the web for documentation" },
];

// The agent chooses tools based on the task
// Task: "Implement the feature described in LINEAR-123"
// Agent: "I need to read the issue first" → calls linear_get_issue
// Agent: "Now I'll implement it" → calls edit_file
// Agent: "Let me test against the DB" → calls postgres_query
// Agent: "Create a PR" → calls github_create_pr
```

## An MCP-Powered Agent Workflow

Here's a complete workflow showing how MCP tools enable end-to-end automation:

```
┌────────────────────────────────────────────────────────────────┐
│          MCP-POWERED FEATURE IMPLEMENTATION                     │
│                                                                │
│  1. READ SPEC                                                  │
│     ┌────────────────────────────────────┐                    │
│     │ MCP: linear_get_issue("FE-456")    │                    │
│     │ → Returns: "Add user avatar upload │                    │
│     │   with S3 storage, 5MB max..."     │                    │
│     └────────────────────────────────────┘                    │
│                        │                                       │
│  2. RESEARCH                                                   │
│     ┌────────────────────────────────────┐                    │
│     │ MCP: web_search("S3 presigned      │                    │
│     │   upload TypeScript best practices")│                    │
│     │ → Returns: relevant documentation   │                    │
│     └────────────────────────────────────┘                    │
│                        │                                       │
│  3. CHECK EXISTING DATA                                        │
│     ┌────────────────────────────────────┐                    │
│     │ MCP: postgres_query("SELECT        │                    │
│     │   column_name FROM                 │                    │
│     │   information_schema.columns       │                    │
│     │   WHERE table_name = 'users'")     │                    │
│     │ → Returns: existing user columns    │                    │
│     └────────────────────────────────────┘                    │
│                        │                                       │
│  4. IMPLEMENT (built-in file tools)                            │
│     └── Write code, create migration, add tests               │
│                        │                                       │
│  5. TEST (built-in shell tool)                                 │
│     └── Run test suite                                        │
│                        │                                       │
│  6. CREATE PR                                                  │
│     ┌────────────────────────────────────┐                    │
│     │ MCP: github_create_pr({            │                    │
│     │   title: "Add avatar upload",      │                    │
│     │   body: "Implements FE-456..."     │                    │
│     │ })                                 │                    │
│     └────────────────────────────────────┘                    │
│                        │                                       │
│  7. UPDATE TRACKER                                             │
│     ┌────────────────────────────────────┐                    │
│     │ MCP: linear_update_issue("FE-456", │                    │
│     │   { status: "In Review" })          │                    │
│     └────────────────────────────────────┘                    │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

## Configuring MCP Tools

### Copilot CLI Configuration

```json
// .vscode/mcp.json or ~/.copilot/mcp-config.json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "DATABASE_URL": "${DATABASE_URL}"
      }
    },
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/workspace"]
    }
  }
}
```

### Allowing MCP Tools

```bash
# Allow all tools from a specific MCP server
copilot-cli --allow-tool='github(*)'

# Allow specific MCP tools
copilot-cli --allow-tool='github(create_pull_request)'
copilot-cli --allow-tool='postgres(query)'

# Allow MCP tools alongside built-in tools
copilot-cli --allow-tool='shell(git)' --allow-tool='github(create_pull_request)'
```

## Implementation: MCP-Aware Agent

```typescript
interface MCPTool {
  name: string;
  description: string;
  inputSchema: JSONSchema;
  serverName: string;
}

async function mcpAwareAgent(task: string) {
  // Discover available MCP tools
  const mcpTools = await discoverMCPTools();
  const builtinTools = [shell, readFile, editFile, search];
  const allTools = [...builtinTools, ...mcpTools];

  console.log(`Agent has ${builtinTools.length} built-in + ${mcpTools.length} MCP tools`);

  // Agent selects tools based on task
  return await agentLoop({
    task,
    tools: allTools,
    system: `You have access to both built-in tools and MCP-connected services.
             Use MCP tools when you need external data or integrations.
             Prefer built-in tools for file operations and shell commands.`
  });
}

async function discoverMCPTools(): Promise<MCPTool[]> {
  const tools: MCPTool[] = [];

  for (const [serverName, config] of Object.entries(mcpConfig.servers)) {
    const client = await connectMCPServer(config);
    const serverTools = await client.listTools();

    for (const tool of serverTools) {
      tools.push({
        ...tool,
        serverName,
        // Prefix tool name with server for clarity
        name: `${serverName}_${tool.name}`
      });
    }
  }

  return tools;
}
```

### Python Implementation

```python
import json
from dataclasses import dataclass

@dataclass
class MCPToolCall:
    server: str
    tool: str
    arguments: dict

async def mcp_agent(task: str, mcp_servers: dict) -> str:
    # Connect to all configured MCP servers
    tools = []
    for server_name, config in mcp_servers.items():
        client = await connect_mcp(config)
        server_tools = await client.list_tools()
        for tool in server_tools:
            tools.append({
                "name": f"{server_name}_{tool['name']}",
                "description": tool["description"],
                "server": server_name,
                "schema": tool.get("inputSchema", {})
            })

    # Agent loop with MCP tools available
    history = []
    for step in range(20):
        response = await llm.complete(
            system="You are a coding agent with access to external services via MCP.",
            tools=tools,
            prompt=f"Task: {task}\nHistory: {json.dumps(history[-5:])}"
        )

        if response.tool_call:
            # Route to the appropriate MCP server
            server = response.tool_call.server
            result = await mcp_clients[server].call_tool(
                response.tool_call.tool,
                response.tool_call.arguments
            )
            history.append({"action": response.tool_call, "result": result})
        else:
            return response.text

    return "Max steps reached"
```

## Key Considerations

### Token Cost of Tool Descriptions

Every MCP tool's description consumes tokens in the agent's context window. With many tools, this can become significant.

```
┌────────────────────────────────────────────────────────────────┐
│              TOOL DESCRIPTION TOKEN COST                        │
│                                                                │
│  5 built-in tools    ~500 tokens                               │
│  + 10 MCP tools      ~2,000 tokens                             │
│  + 20 MCP tools      ~5,000 tokens   ← starts to add up       │
│  + 50 MCP tools      ~12,000 tokens  ← significant cost        │
│                                                                │
│  Strategies:                                                   │
│  1. Only connect MCP servers needed for current task           │
│  2. Use concise tool descriptions                              │
│  3. Group related tools under fewer MCP servers                │
│  4. Consider dynamic tool loading (load on demand)             │
└────────────────────────────────────────────────────────────────┘
```

### Too Many Tools Confuses the Agent

Research shows that agent performance can **degrade** with too many tools — the model struggles to select the right one.

```typescript
// ❌ BAD: 50 tools — agent gets confused
const tools = [...builtinTools, ...allMCPTools]; // 50 tools

// ✅ GOOD: Curate tools for the task
const tools = [
  ...builtinTools,
  ...selectRelevantMCPTools(task, allMCPTools, maxTools: 10)
];

function selectRelevantMCPTools(task: string, tools: MCPTool[], maxTools: number): MCPTool[] {
  // Use a lightweight LLM call to select relevant tools
  const selection = await llm.complete({
    prompt: `Given task: "${task}"
             Which of these tools might be needed?
             ${tools.map(t => `${t.name}: ${t.description}`).join("\n")}
             Return the names of up to ${maxTools} relevant tools.`
  });

  const selectedNames = parseToolNames(selection);
  return tools.filter(t => selectedNames.includes(t.name));
}
```

### Error Handling for MCP Tools

MCP tools can fail in ways built-in tools don't — network errors, auth failures, rate limits:

```typescript
async function callMCPToolSafely(
  server: MCPClient, 
  tool: string, 
  args: Record<string, unknown>
): Promise<ToolResult> {
  try {
    const result = await withTimeout(
      server.callTool(tool, args),
      10_000 // 10s timeout for MCP calls
    );
    return { success: true, data: result };
  } catch (error) {
    if (error.code === "AUTH_FAILED") {
      return { 
        success: false, 
        error: `Authentication failed for ${server.name}. Check credentials.` 
      };
    }
    if (error.code === "RATE_LIMITED") {
      return { 
        success: false, 
        error: `Rate limited by ${server.name}. Wait and retry.`,
        retryAfter: error.retryAfter 
      };
    }
    return { 
      success: false, 
      error: `MCP tool ${tool} failed: ${error.message}. Try alternative approach.` 
    };
  }
}
```
