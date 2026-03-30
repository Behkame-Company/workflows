# Custom Tool Development

> Building custom tools for your AI coding agents — from simple shell scripts to full MCP servers — so agents can perform project-specific tasks reliably.

---

## Why Build Custom Tools?

Out-of-the-box tools (shell, file edit, search) handle generic tasks. But your project has **specific workflows** that agents repeat over and over:

- Running your particular test suite with the right environment variables
- Deploying to your staging environment
- Querying your internal API in the correct format
- Applying your team's code formatting standards

Custom tools encode these workflows so the agent doesn't have to figure them out each time.

```
┌──────────────────────────────────────────────────────────────────┐
│                CUSTOM TOOL SPECTRUM                               │
│                                                                  │
│  Simple                                                Complex  │
│  ◄──────────────────────────────────────────────────────────►   │
│                                                                  │
│  ┌──────────┐  ┌──────────────┐  ┌────────────┐  ┌───────────┐│
│  │  Shell    │  │  Copilot     │  │  CLAUDE.md │  │   MCP     ││
│  │  Scripts  │  │  Skills      │  │  Commands  │  │  Servers  ││
│  │          │  │              │  │            │  │           ││
│  │  Agent    │  │  Folders of  │  │  Custom    │  │  Standard ││
│  │  runs via │  │  instructions│  │  workflows │  │  protocol ││
│  │  shell    │  │  + scripts   │  │  defined   │  │  works    ││
│  │  tool     │  │              │  │  in config │  │  across   ││
│  │          │  │              │  │            │  │  tools    ││
│  └──────────┘  └──────────────┘  └────────────┘  └───────────┘│
│                                                                  │
│  Effort:   Minutes     Hours        Hours         Days          │
│  Reuse:    This agent  This project This project  Any agent     │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

## Option 1: Shell Scripts (Simplest)

The agent runs custom scripts via its shell tool. No special integration needed.

```bash
#!/bin/bash
# scripts/run-tests.sh — Custom test runner with environment setup

set -euo pipefail

# Set up test environment
export DATABASE_URL="postgresql://localhost:5432/testdb"
export REDIS_URL="redis://localhost:6379/1"
export NODE_ENV="test"

# Reset test database
echo "Resetting test database..."
npx prisma db push --force-reset --skip-generate 2>/dev/null

# Seed test data
echo "Seeding test data..."
npx tsx scripts/seed-test-data.ts

# Run tests with coverage
echo "Running tests..."
npx jest "$@" --coverage --verbose 2>&1

echo "Test run complete."
```

```bash
#!/bin/bash
# scripts/deploy-staging.sh — Deploy to staging with safety checks

set -euo pipefail

BRANCH=$(git rev-parse --abbrev-ref HEAD)
echo "Deploying branch: $BRANCH"

# Safety checks
if [ "$BRANCH" = "main" ]; then
  echo "ERROR: Deploy staging from a feature branch, not main"
  exit 1
fi

# Build and deploy
npm run build
npm run lint
npm test

echo "All checks passed. Deploying to staging..."
aws ecs update-service --cluster staging --service api --force-new-deployment

echo "Deployed. Health check: https://staging-api.example.com/health"
```

The agent uses these naturally:

```
Agent thought: "I need to run the project's tests"
Agent action: shell("bash scripts/run-tests.sh --grep 'auth'")
Agent observation: "12 tests passed, 0 failed"
```

## Option 2: Copilot Skills (Folders of Instructions)

Skills are folders containing instructions, scripts, and resources that Copilot CLI agents can discover and use.

```
.github/
└── skills/
    └── run-e2e-tests/
        ├── instructions.md     # What this skill does, when to use it
        ├── run-tests.sh        # The actual test runner
        └── test-config.json    # Configuration the script needs
```

```markdown
<!-- .github/skills/run-e2e-tests/instructions.md -->
# Run E2E Tests

Use this skill when you need to run end-to-end tests for the project.

## When to Use
- After implementing a new feature
- After modifying API routes
- Before creating a pull request

## How to Run
Execute `bash .github/skills/run-e2e-tests/run-tests.sh`

## Environment Requirements
- Docker must be running (for test database)
- Port 3000 must be free (test server)

## Interpreting Results
- Tests output TAP format
- "not ok" lines indicate failures
- Screenshots of failures are saved to `test-results/`

## Common Issues
- "Connection refused": Docker isn't running. Start with `docker compose up -d`
- "Port in use": Kill the process on port 3000 first
```

## Option 3: CLAUDE.md Custom Commands

For Claude Code, define custom workflows in CLAUDE.md:

```markdown
<!-- CLAUDE.md -->
## Custom Commands

### /test [filter]
Run the project test suite:
1. Start Docker services: `docker compose up -d postgres redis`
2. Wait for health checks: `scripts/wait-for-services.sh`
3. Run tests: `npm test -- --grep "${filter:-.*}"`
4. Report coverage: parse coverage/lcov-report/index.html for summary

### /deploy-preview
Create a preview deployment:
1. Build: `npm run build`
2. Deploy: `npx vercel --yes`
3. Run smoke tests against preview URL
4. Report preview URL

### /db-migrate
Run database migrations:
1. Generate: `npx prisma migrate dev --name "${description}"`
2. Verify: `npx prisma validate`
3. Update types: `npx prisma generate`
```

## Option 4: MCP Servers (Standard Protocol)

For tools that should work across different AI coding agents, build an MCP server.

### Step-by-Step MCP Server

```typescript
// src/my-tools-server.ts
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";

const server = new McpServer({
  name: "project-tools",
  version: "1.0.0"
});

// Tool 1: Run project-specific tests
server.tool(
  "run_tests",
  `Run the project test suite with proper environment setup.
   
   Parameters:
   - filter: Optional regex to filter tests (e.g., "auth" runs auth tests)
   - coverage: Whether to generate coverage report (default: false)
   
   Returns test results with pass/fail counts and any error details.`,
  {
    filter: z.string().optional().describe("Regex to filter tests"),
    coverage: z.boolean().optional().describe("Generate coverage report")
  },
  async ({ filter, coverage }) => {
    const env = {
      ...process.env,
      DATABASE_URL: "postgresql://localhost:5432/testdb",
      NODE_ENV: "test"
    };

    const args = [
      "npx", "jest",
      filter ? `--grep="${filter}"` : "",
      coverage ? "--coverage" : "",
      "--verbose"
    ].filter(Boolean).join(" ");

    const result = await exec(args, { env });

    return {
      content: [{
        type: "text",
        text: JSON.stringify({
          passed: result.exitCode === 0,
          output: result.stdout,
          errors: result.stderr,
          summary: extractTestSummary(result.stdout)
        })
      }]
    };
  }
);

// Tool 2: Query project database
server.tool(
  "query_db",
  `Query the project database. Read-only — SELECT queries only.
   
   Parameters:
   - sql: The SQL query to execute (SELECT only)
   - database: Which database to query ("development" or "test")
   
   Returns query results as JSON rows.`,
  {
    sql: z.string().describe("SQL SELECT query"),
    database: z.enum(["development", "test"]).default("development")
  },
  async ({ sql, database }) => {
    // Safety: only allow SELECT
    if (!/^\s*SELECT/i.test(sql)) {
      return {
        content: [{ type: "text", text: "ERROR: Only SELECT queries allowed" }],
        isError: true
      };
    }

    const dbUrl = database === "test"
      ? process.env.TEST_DATABASE_URL
      : process.env.DATABASE_URL;

    const rows = await queryDatabase(dbUrl, sql);

    return {
      content: [{ type: "text", text: JSON.stringify(rows, null, 2) }]
    };
  }
);

// Tool 3: Get project-specific context
server.tool(
  "get_architecture",
  `Get an overview of the project architecture.
   Returns the key directories, their purposes, and important files.
   Use this as a starting point when exploring the codebase.`,
  {},
  async () => {
    const arch = await fs.readFile("docs/ARCHITECTURE.md", "utf-8");
    return {
      content: [{ type: "text", text: arch }]
    };
  }
);

// Start the server
const transport = new StdioServerTransport();
await server.connect(transport);
```

### Register the MCP Server

```json
// .vscode/mcp.json
{
  "mcpServers": {
    "project-tools": {
      "command": "npx",
      "args": ["tsx", "src/my-tools-server.ts"],
      "env": {
        "DATABASE_URL": "${DATABASE_URL}"
      }
    }
  }
}
```

## The Design Process

```
┌────────────────────────────────────────────────────────────────┐
│              CUSTOM TOOL DESIGN PROCESS                          │
│                                                                │
│  1. IDENTIFY                                                   │
│     Watch the agent work. What tasks does it:                  │
│     • Repeat frequently?                                       │
│     • Get wrong consistently?                                  │
│     • Spend many steps figuring out?                           │
│                                                                │
│  2. DESIGN                                                     │
│     Define the tool interface:                                 │
│     • What inputs does the agent need to provide?              │
│     • What output format helps the agent most?                 │
│     • What errors are possible and how to report them?         │
│                                                                │
│  3. IMPLEMENT                                                  │
│     Build it — start with shell script, upgrade if needed:     │
│     • Shell script → Copilot skill → MCP server               │
│                                                                │
│  4. TEST WITH AGENT                                            │
│     Have the agent use it on real tasks:                       │
│     • Does the agent choose it at the right time?              │
│     • Does the agent pass correct inputs?                      │
│     • Does the output help the agent proceed?                  │
│                                                                │
│  5. ITERATE                                                    │
│     Based on agent usage patterns, improve:                    │
│     • Better descriptions                                      │
│     • More helpful error messages                              │
│     • Input validation (poka-yoke)                             │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

### Example: Iterating on Tool Design

```typescript
// v1: Agent often passes wrong database name
const tool_v1 = {
  name: "query_db",
  description: "Query the database",
  params: { sql: "string", db: "string" }
};

// v2: Constrain to valid values, add examples
const tool_v2 = {
  name: "query_db",
  description: `Query the project database (read-only).
    
    Parameters:
    - sql: SELECT query to run
    - db: "development" | "test" (defaults to "development")
    
    Example: query_db({ sql: "SELECT * FROM users LIMIT 5", db: "development" })`,
  params: { 
    sql: "string", 
    db: { enum: ["development", "test"], default: "development" } 
  }
};

// v3: Add safety rails after agent tried a DELETE
const tool_v3 = {
  // ... same description as v2, plus:
  validate: (params) => {
    if (!/^\s*SELECT/i.test(params.sql)) {
      return "Only SELECT queries allowed. Use shell tool for migrations.";
    }
    if (/;\s*\S/.test(params.sql)) {
      return "Only one statement allowed per call.";
    }
    return null;
  }
};
```

## Python MCP Server Example

```python
# my_tools/server.py
from mcp.server import Server
from mcp.server.stdio import stdio_server
import subprocess
import json

app = Server("project-tools")

@app.tool()
async def run_tests(filter: str = "", coverage: bool = False) -> str:
    """Run the project test suite with proper environment setup.
    
    Args:
        filter: Optional regex to filter tests (e.g., "auth")
        coverage: Generate coverage report (default: false)
    """
    cmd = ["python", "-m", "pytest", "-v"]
    if filter:
        cmd.extend(["-k", filter])
    if coverage:
        cmd.extend(["--cov=src", "--cov-report=term-missing"])

    result = subprocess.run(cmd, capture_output=True, text=True, timeout=120)

    return json.dumps({
        "passed": result.returncode == 0,
        "output": result.stdout[-2000:],  # Truncate to avoid context overflow
        "errors": result.stderr[-1000:] if result.returncode != 0 else "",
    })

@app.tool()
async def lint_and_format(path: str = "src/") -> str:
    """Run linter and formatter on the specified path.
    
    Args:
        path: Directory or file to lint (default: "src/")
    """
    results = {}

    # Run ruff (linter)
    lint = subprocess.run(
        ["ruff", "check", path, "--output-format=json"],
        capture_output=True, text=True
    )
    results["lint_issues"] = json.loads(lint.stdout) if lint.stdout else []

    # Run ruff format check
    fmt = subprocess.run(
        ["ruff", "format", "--check", path],
        capture_output=True, text=True
    )
    results["format_ok"] = fmt.returncode == 0

    return json.dumps(results)

async def main():
    async with stdio_server() as (read_stream, write_stream):
        await app.run(read_stream, write_stream)

if __name__ == "__main__":
    import asyncio
    asyncio.run(main())
```

## Decision Guide: Which Approach?

| Approach | Best For | Effort | Reusability |
|----------|----------|--------|-------------|
| **Shell scripts** | Quick, project-specific tasks | Minutes | This project |
| **Copilot skills** | Documented workflows with instructions | Hours | Copilot CLI |
| **CLAUDE.md commands** | Claude Code custom workflows | Hours | Claude Code |
| **MCP servers** | Cross-tool capabilities, team sharing | Days | Any MCP client |

**Rule of thumb**: Start with a shell script. If the agent uses it successfully but the setup is complex, upgrade to a skill or MCP server.
