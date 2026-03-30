# The Augmented LLM

> The fundamental building block of every agentic system: an LLM enhanced with retrieval, tools, and memory that can actively search, act, and remember.

---

## The Core Concept

Before building workflows or agents, you need a solid **augmented LLM** — the basic building block that all agentic patterns are composed from.

An augmented LLM goes beyond simple prompt-in/text-out. It's an LLM enhanced with three capabilities:

1. **Retrieval** — pulling in relevant context from external sources
2. **Tools** — taking actions in the real world
3. **Memory** — persisting information across interactions

```
┌──────────────────────────────────────────────────────────────┐
│                     THE AUGMENTED LLM                        │
│                                                              │
│                    ┌─────────────┐                           │
│     Retrieval      │             │      Tools                │
│   ┌──────────┐    │             │    ┌──────────┐           │
│   │ Codebase │◄───│             │───►│  File    │           │
│   │   RAG    │    │             │    │  System  │           │
│   ├──────────┤    │     LLM     │    ├──────────┤           │
│   │   Docs   │◄───│             │───►│  Shell   │           │
│   │  Search  │    │             │    │Commands  │           │
│   ├──────────┤    │             │    ├──────────┤           │
│   │ Web/API  │◄───│             │───►│   Git    │           │
│   │  Fetch   │    │             │    │Operations│           │
│   └──────────┘    │             │    ├──────────┤           │
│                    │             │───►│   MCP    │           │
│     Memory         │             │    │  Tools   │           │
│   ┌──────────┐    │             │    └──────────┘           │
│   │CLAUDE.md │◄──►│             │                           │
│   ├──────────┤    │             │                           │
│   │ Session  │◄──►│             │                           │
│   │ Context  │    └─────────────┘                           │
│   └──────────┘                                              │
└──────────────────────────────────────────────────────────────┘
```

The critical insight from Anthropic's research:

> *Current models can actively use these capabilities — generating their own search queries, selecting appropriate tools, and determining what information to retain.*

The LLM isn't passively receiving augmentation. It **drives** the augmentation — deciding when to search, which tools to call, and what to remember.

## The Three Pillars

### 1. Retrieval: Bringing in Context

The model doesn't have your codebase in its weights. Retrieval bridges that gap.

```typescript
// Retrieval strategies for coding agents
const retrievalSources = {
  // Semantic search over codebase
  codebaseRAG: async (query: string) => {
    return await vectorStore.search(query, { topK: 10 });
  },
  
  // Direct file access
  fileSystem: async (path: string) => {
    return await fs.readFile(path, 'utf-8');
  },
  
  // Documentation lookup
  docs: async (topic: string) => {
    return await docIndex.search(topic);
  },
  
  // Git history for context
  gitHistory: async (file: string) => {
    return await git.log({ file, maxCount: 10 });
  }
};
```

**Key design principle**: Tailor retrieval to your specific use case. A coding agent needs different retrieval than a customer support agent. Consider:

- What context does the model need most often?
- What's the freshness requirement? (live file system vs. indexed snapshots)
- How much context can you afford per call? (token budget)

### 2. Tools: Taking Action

Tools let the model **do things** — not just think about them.

```typescript
// Core tools for a coding-focused augmented LLM
const codingTools = [
  {
    name: "read_file",
    description: "Read the contents of a file at the given absolute path",
    parameters: { path: "string — absolute path to the file" },
    execute: async ({ path }) => fs.readFile(path, 'utf-8')
  },
  {
    name: "write_file", 
    description: "Write content to a file, creating it if it doesn't exist",
    parameters: { 
      path: "string — absolute path",
      content: "string — full file content" 
    },
    execute: async ({ path, content }) => fs.writeFile(path, content)
  },
  {
    name: "run_command",
    description: "Execute a shell command and return stdout/stderr",
    parameters: { command: "string — the command to run" },
    execute: async ({ command }) => exec(command)
  },
  {
    name: "search_code",
    description: "Search for a pattern across all files using ripgrep",
    parameters: { pattern: "string — regex pattern to search for" },
    execute: async ({ pattern }) => exec(`rg '${pattern}' --json`)
  }
];
```

**Key design principle**: Ensure an easy, well-documented interface for your LLM. Tool descriptions are prompts — they directly affect how well the model uses the tool. See [Agent-Computer Interface Design](./agent-computer-interface.md) for a deep dive.

### 3. Memory: Persisting Knowledge

Memory lets the model carry context across interactions and sessions.

```
┌──────────────────────────────────────────────────┐
│                  MEMORY LAYERS                    │
│                                                   │
│  ┌─────────────────────────────────────────────┐ │
│  │ Session Memory (conversation context)       │ │
│  │ "I already changed auth.ts in this session" │ │
│  ├─────────────────────────────────────────────┤ │
│  │ Project Memory (CLAUDE.md / instructions)   │ │
│  │ "This project uses Vitest, not Jest"        │ │
│  ├─────────────────────────────────────────────┤ │
│  │ Persistent Memory (learned preferences)     │ │
│  │ "User prefers functional style over OOP"    │ │
│  └─────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────┘
```

In coding contexts, memory takes several forms:

| Memory Type | Mechanism | Example |
|---|---|---|
| **Instruction files** | `CLAUDE.md`, `copilot-instructions.md` | "Use single quotes, tabs for indentation" |
| **Session context** | Conversation history | "We already fixed the auth bug" |
| **Project knowledge** | README, docs, architecture files | "The API uses REST, not GraphQL" |
| **Learned memory** | Persistent storage across sessions | "This user prefers concise explanations" |

## The Augmented LLM in Practice: Coding Agents

A modern coding agent like GitHub Copilot CLI is a textbook augmented LLM:

```
┌────────────────────────────────────────────────────────┐
│           CODING AGENT = AUGMENTED LLM                 │
│                                                        │
│  Model:    Claude / GPT / etc.                         │
│                                                        │
│  Retrieval:                                            │
│    • File system access (read any project file)        │
│    • Grep/glob search across codebase                  │
│    • Git history and diff inspection                   │
│    • Web fetch for documentation                       │
│                                                        │
│  Tools:                                                │
│    • File read/write/edit                              │
│    • Shell command execution                           │
│    • Git operations (commit, branch, diff)             │
│    • MCP servers (database, API, custom tools)         │
│    • Sub-agent spawning                                │
│                                                        │
│  Memory:                                               │
│    • CLAUDE.md / copilot-instructions.md               │
│    • Session conversation history                      │
│    • Stored memories across sessions                   │
│    • Project documentation (README, docs/)             │
└────────────────────────────────────────────────────────┘
```

## MCP: The Integration Standard

The **Model Context Protocol (MCP)** is emerging as the standard way to connect augmented LLMs to external systems. Instead of building custom integrations for every tool, MCP provides a uniform interface:

```jsonc
// .vscode/mcp.json — connecting tools via MCP
{
  "servers": {
    "database": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": { "DATABASE_URL": "postgresql://localhost/mydb" }
    },
    "github": {
      "command": "npx", 
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": { "GITHUB_TOKEN": "${GITHUB_TOKEN}" }
    }
  }
}
```

MCP matters because it standardizes the "tool" pillar — any MCP-compatible tool works with any MCP-compatible agent, reducing the integration burden.

## Two Key Design Principles

Anthropic emphasizes two principles when building augmented LLMs:

### 1. Tailor Capabilities to Your Use Case

Don't give your model every tool imaginable. A focused set of well-designed tools outperforms a sprawling toolkit:

```typescript
// ❌ Too many unfocused tools
const tools = [readFile, writeFile, exec, search, git, docker, 
               kubernetes, aws, gcp, azure, slack, email, jira, ...];

// ✅ Focused tools for the task
const codeReviewTools = [readFile, search, gitDiff, gitLog, addComment];
```

### 2. Ensure Easy, Well-Documented Interfaces

Your tool descriptions are the "UX" for the model. Invest in them:

```typescript
// ❌ Minimal description
{ name: "search", description: "Search files" }

// ✅ Rich, clear description
{ 
  name: "search_code",
  description: `Search for a regex pattern across project files using ripgrep.
    Returns matching lines with file paths and line numbers.
    Use this to find function definitions, usages, or patterns.
    Example: search_code({ pattern: "async function handle" })
    Note: Searches from project root. Use glob parameter to filter file types.`
}
```

## Building Your Own Augmented LLM

A minimal but complete augmented LLM for coding tasks:

```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();

const tools = [
  {
    name: "read_file",
    description: "Read a file's contents. Returns the full text of the file at the given path.",
    input_schema: {
      type: "object",
      properties: { path: { type: "string", description: "Absolute file path" } },
      required: ["path"]
    }
  },
  {
    name: "list_directory",
    description: "List files and subdirectories at the given path. Use to explore project structure.",
    input_schema: {
      type: "object",
      properties: { path: { type: "string", description: "Absolute directory path" } },
      required: ["path"]
    }
  },
  {
    name: "run_command",
    description: "Run a shell command. Use for git operations, tests, builds, or file searches.",
    input_schema: {
      type: "object",
      properties: { command: { type: "string", description: "Shell command to execute" } },
      required: ["command"]
    }
  }
];

// The augmented LLM: model + tools + memory (system prompt)
async function augmentedLLM(userMessage: string, projectMemory: string) {
  return await client.messages.create({
    model: "claude-sonnet-4-20250514",
    max_tokens: 4096,
    system: projectMemory,  // Memory: project context and conventions
    tools,                   // Tools: file system, shell, search
    messages: [{ role: "user", content: userMessage }]
  });
}
```
