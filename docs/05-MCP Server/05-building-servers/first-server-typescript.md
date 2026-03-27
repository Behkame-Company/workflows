# Your First MCP Server (TypeScript)

> Build a working MCP server in TypeScript step-by-step using the official SDK.

---

## Prerequisites

- Node.js 18+
- TypeScript knowledge
- A terminal

---

## Step 1: Set Up the Project

```bash
mkdir my-mcp-server && cd my-mcp-server
npm init -y
npm install @modelcontextprotocol/sdk
npm install -D typescript @types/node tsx
npx tsc --init
```

Update `tsconfig.json`:
```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "Node16",
    "moduleResolution": "Node16",
    "outDir": "./dist",
    "strict": true,
    "esModuleInterop": true
  }
}
```

---

## Step 2: Create the Server

Create `src/server.ts`:

```typescript
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import {
  CallToolRequestSchema,
  ListToolsRequestSchema,
  ListResourcesRequestSchema,
  ReadResourceRequestSchema,
} from "@modelcontextprotocol/sdk/types.js";

// Create server
const server = new Server(
  { name: "my-first-server", version: "1.0.0" },
  { capabilities: { tools: {}, resources: {} } }
);

// ── Define Tools ──
server.setRequestHandler(ListToolsRequestSchema, async () => ({
  tools: [
    {
      name: "add_numbers",
      description: "Add two numbers together. Use for arithmetic addition.",
      inputSchema: {
        type: "object" as const,
        properties: {
          a: { type: "number", description: "First number" },
          b: { type: "number", description: "Second number" },
        },
        required: ["a", "b"],
      },
    },
    {
      name: "get_weather",
      description: "Get current weather for a city (mock data for demo).",
      inputSchema: {
        type: "object" as const,
        properties: {
          city: { type: "string", description: "City name" },
        },
        required: ["city"],
      },
    },
  ],
}));

// ── Handle Tool Calls ──
server.setRequestHandler(CallToolRequestSchema, async (request) => {
  const { name, arguments: args } = request.params;

  switch (name) {
    case "add_numbers": {
      const { a, b } = args as { a: number; b: number };
      return {
        content: [{ type: "text", text: `${a} + ${b} = ${a + b}` }],
      };
    }
    case "get_weather": {
      const { city } = args as { city: string };
      return {
        content: [{
          type: "text",
          text: `Weather in ${city}: 72°F, Sunny, Humidity: 45%`,
        }],
      };
    }
    default:
      throw new Error(`Unknown tool: ${name}`);
  }
});

// ── Define Resources ──
server.setRequestHandler(ListResourcesRequestSchema, async () => ({
  resources: [
    {
      uri: "config://app/info",
      name: "Application Info",
      mimeType: "text/plain",
    },
  ],
}));

server.setRequestHandler(ReadResourceRequestSchema, async (request) => {
  if (request.params.uri === "config://app/info") {
    return {
      contents: [{
        uri: "config://app/info",
        mimeType: "text/plain",
        text: "App: My First MCP Server\nVersion: 1.0.0\nRuntime: Node.js",
      }],
    };
  }
  throw new Error(`Unknown resource: ${request.params.uri}`);
});

// ── Start Server ──
async function main() {
  const transport = new StdioServerTransport();
  await server.connect(transport);
  console.error("MCP server running on stdio");
}

main().catch(console.error);
```

---

## Step 3: Run and Test

### Run Directly
```bash
npx tsx src/server.ts
```

### With Claude Desktop
```json
{
  "mcpServers": {
    "my-server": {
      "command": "npx",
      "args": ["tsx", "/path/to/my-mcp-server/src/server.ts"]
    }
  }
}
```

### With VS Code
```json
{
  "mcp": {
    "servers": {
      "my-server": {
        "command": "npx",
        "args": ["tsx", "${workspaceFolder}/src/server.ts"]
      }
    }
  }
}
```

---

## Step 4: Build for Production

```bash
npx tsc
node dist/server.js
```

Add to `package.json`:
```json
{
  "bin": { "my-mcp-server": "dist/server.js" },
  "scripts": {
    "build": "tsc",
    "start": "node dist/server.js",
    "dev": "tsx src/server.ts"
  }
}
```

---

## Python vs TypeScript Comparison

| Aspect | Python (FastMCP) | TypeScript (SDK) |
|--------|-----------------|------------------|
| **Boilerplate** | Minimal (decorators) | More verbose (handlers) |
| **Type safety** | Runtime only | Compile-time |
| **Tool definition** | `@mcp.tool` decorator | Schema objects + handler |
| **Learning curve** | Lower | Moderate |
| **npm distribution** | Not applicable | `npx` ready |
| **Best for** | Quick prototyping, data science | Production services, npm ecosystem |

---

## Key Takeaways

1. TypeScript servers use **schema objects** and **request handlers** (more explicit than Python decorators)
2. Use `tsx` for development, `tsc` + `node` for production
3. Log to **stderr** (`console.error`), never stdout
4. Handle **unknown tools/resources** gracefully with error messages
5. TypeScript's type system catches schema mismatches at compile time

---

## Next Steps

- 🔗 [Tool Design Patterns](tool-design-patterns.md) — Design tools AI agents love
- 🔗 [Testing & Debugging](testing-and-debugging.md) — Validation strategies
