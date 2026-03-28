# Agent-Computer Interface (ACI) Design

> Invest as much effort in your agent-computer interface as you would in a human-computer interface. The quality of your tool definitions directly determines how effectively an LLM can use them.

---

## The Core Principle

From Anthropic's "Prompt Engineering your Tools" (Appendix 2):

> *"Just as a human-computer interface (HCI) can make or break a user's experience, an agent-computer interface (ACI) can dramatically impact agent performance."*

Tool design is **prompt engineering for actions**. Every tool name, description, and parameter schema is a prompt that tells the model what the tool does, when to use it, and how to call it correctly.

## The Five Principles of ACI Design

### 1. Put Yourself in the Model's Shoes

Before shipping a tool, ask: **"If I saw this tool description with no other context, would I know exactly when and how to use it?"**

```typescript
// ❌ The model's perspective: "What does this do? When would I use it?"
{
  name: "exec",
  description: "Execute something",
  parameters: { input: "string" }
}

// ✅ The model's perspective: "I know exactly what this does and when to use it"
{
  name: "run_shell_command",
  description: `Execute a shell command in the project's root directory and return 
    stdout and stderr. Use this to run tests (npm test), check types (npx tsc --noEmit), 
    install packages (npm install X), or run any CLI tool. The command runs in a bash 
    shell with a 30-second timeout.`,
  parameters: { 
    command: {
      type: "string",
      description: "The shell command to execute, e.g., 'npm test' or 'git status'"
    }
  }
}
```

### 2. Include Examples, Edge Cases, and Format Requirements

Good tool descriptions are like great docstrings — they cover the happy path, edge cases, and input format:

```typescript
{
  name: "edit_file",
  description: `Make a targeted edit to a file by replacing an exact string match.
    
    IMPORTANT:
    - old_str must match EXACTLY one occurrence in the file (including whitespace)
    - If old_str appears multiple times, the edit will fail — add more surrounding 
      context to make it unique
    - new_str replaces the entire old_str match
    - To delete code, set new_str to an empty string
    
    Example:
      edit_file({
        path: "/project/src/utils.ts",
        old_str: "const MAX_RETRIES = 3;",
        new_str: "const MAX_RETRIES = 5;"
      })
    
    Edge case — if the string has special characters or indentation, 
    include them exactly as they appear in the file.`,
  parameters: {
    path: { type: "string", description: "Absolute path to the file to edit" },
    old_str: { type: "string", description: "Exact string to find and replace (must be unique in file)" },
    new_str: { type: "string", description: "Replacement string (empty string to delete)" }
  }
}
```

### 3. Optimize Parameter Names and Descriptions

Parameter names are tokens the model processes. Make them self-documenting:

```typescript
// ❌ Ambiguous parameter names
{
  name: "search",
  parameters: {
    q: "string",          // What kind of query? Text? Regex? Glob?
    t: "string",          // What is 't'?
    n: "number"           // Number of what?
  }
}

// ✅ Self-documenting parameter names
{
  name: "search_code",
  parameters: {
    pattern: {
      type: "string",
      description: "Regex pattern to search for (uses ripgrep syntax)"
    },
    file_type: {
      type: "string", 
      description: "File extension filter, e.g., 'ts', 'py', 'go'. Omit to search all files."
    },
    max_results: {
      type: "number",
      description: "Maximum results to return (default: 20). Use lower values for broad patterns."
    }
  }
}
```

### 4. Test and Iterate on Model Usage

Don't assume your tool design works — **test it**. Watch how the model actually calls your tools and fix the mistakes:

```
┌──────────────────────────────────────────────────────────────┐
│              ACI DESIGN ITERATION LOOP                       │
│                                                              │
│  1. Define tool ──► 2. Test with model ──► 3. Observe usage │
│        ▲                                          │          │
│        │                                          ▼          │
│  5. Update   ◄──── 4. Identify mistakes/confusion            │
│     design                                                   │
│                                                              │
│  Common issues to watch for:                                 │
│  • Model calls the wrong tool for a task                    │
│  • Model passes wrong parameter format                      │
│  • Model doesn't use a tool when it should                  │
│  • Model uses tool in unexpected/suboptimal ways            │
└──────────────────────────────────────────────────────────────┘
```

Real example from Anthropic's SWE-bench work: they observed models struggling with relative file paths, so they changed the tool to require absolute paths — this single change eliminated a whole class of errors.

### 5. Poka-Yoke: Make Mistakes Structurally Impossible

[Poka-yoke](https://en.wikipedia.org/wiki/Poka-yoke) is a Japanese manufacturing term meaning "mistake-proofing." Apply it to your tools:

```typescript
// ❌ Error-prone: relative paths cause constant confusion
{
  name: "read_file",
  parameters: {
    path: { type: "string", description: "Path to the file" }
    // Model might pass "src/utils.ts" or "./src/utils.ts" or "utils.ts"
  }
}

// ✅ Poka-yoke: force absolute paths, eliminate ambiguity
{
  name: "read_file",
  parameters: {
    path: { 
      type: "string", 
      description: "Absolute path to the file (must start with /). Example: /home/user/project/src/utils.ts" 
    }
  },
  // Runtime validation as extra safety net
  validate: (params) => {
    if (!params.path.startsWith("/")) {
      throw new Error(`Path must be absolute (start with /). Got: ${params.path}`);
    }
  }
}
```

More poka-yoke patterns:

```typescript
// ❌ Error-prone: model has to count line numbers
{
  name: "replace_lines",
  parameters: {
    start_line: "number",  // Models are bad at counting lines
    end_line: "number",
    new_content: "string"
  }
}

// ✅ Poka-yoke: use string matching instead of line counting
{
  name: "edit_file",
  parameters: {
    old_str: "string",  // Exact text to find — no counting needed
    new_str: "string"
  }
}
```

```typescript
// ❌ Error-prone: model has to escape content for JSON
{
  name: "write_file",
  parameters: {
    content: "string"  // What about newlines? Quotes? Backslashes?
  }
}

// ✅ Poka-yoke: handle serialization in the tool, not the model
// The tool accepts the content naturally and handles encoding internally
```

## Before and After: Tool Design Case Study

Here's a real-world example of improving a code search tool:

### Before: Minimal Design

```typescript
const searchTool = {
  name: "search",
  description: "Search the codebase",
  parameters: {
    query: { type: "string" }
  }
};
```

**Problems observed**:
- Model passed natural language queries ("find the authentication logic") instead of patterns
- Model didn't know if it was regex, fuzzy, or exact match
- Model didn't know the scope (whole repo? current directory?)
- Model used it when `read_file` would have been more appropriate

### After: Thoughtful ACI Design

```typescript
const searchTool = {
  name: "search_code",
  description: `Search for a regex pattern across all files in the project repository.
    
    Returns matching lines with file paths and line numbers, formatted as:
    /path/to/file.ts:42: matching line content
    
    USE THIS WHEN: You need to find where something is defined or used across 
    multiple files. For reading a known file, use read_file instead.
    
    PATTERN FORMAT: Uses ripgrep regex syntax. Common patterns:
    - "function handleAuth" — find a function definition
    - "import.*from ['"]express['"]" — find imports
    - "TODO|FIXME|HACK" — find code annotations
    
    TIPS: 
    - Use file_glob to narrow results: "*.ts" for TypeScript only
    - Results are capped at 50 matches. Use a specific pattern if too broad.`,
  parameters: {
    pattern: { 
      type: "string", 
      description: "Regex pattern to search for (ripgrep syntax)" 
    },
    file_glob: { 
      type: "string", 
      description: "Optional glob to filter files, e.g., '*.ts' or 'src/**/*.py'" 
    },
    case_sensitive: {
      type: "boolean",
      description: "Whether the search is case-sensitive (default: true)"
    }
  }
};
```

**Result**: Model uses the tool correctly 95%+ of the time, passes appropriate regex patterns, and uses `read_file` when it already knows the path.

## Tool Format Considerations

Anthropic found several principles about how tools should communicate with the model:

### Give the Model Tokens to Think

Don't force compressed responses. Let the model reason about tool results:

```typescript
// ❌ Forcing structured-only output from tools
{
  name: "analyze_code",
  returns: "JSON object with fields: issues[], suggestions[], score"
  // No room for the model to reason or explain
}

// ✅ Allow natural reasoning alongside structure
{
  name: "analyze_code", 
  returns: `A markdown analysis with:
    1. A brief summary of what the code does
    2. Any issues found (with file:line references)  
    3. Suggested improvements
    The model can explain its reasoning naturally.`
}
```

### Keep the Format Natural

Don't impose formatting overhead that wastes tokens or causes errors:

```typescript
// ❌ Formatting overhead — model has to count lines and track positions
"Edit line 47-52 of file.ts, replacing the content at column 12-38..."

// ✅ Natural format — model works with text naturally
"Replace the exact string 'const x = 1;' with 'const x = 2;' in file.ts"
```

### Minimize String Escaping Requirements

```typescript
// ❌ Model has to manually escape special characters
{ content: "function test() {\n  return \"hello\";\n}" }

// ✅ Tool handles escaping internally — model writes naturally
// The tool implementation handles encoding; the model just provides the content
```

## ACI Design Checklist

Use this checklist when designing or reviewing agent tools:

```
□ Tool name clearly indicates its purpose
□ Description explains WHEN to use (not just WHAT it does)
□ Description includes example usage  
□ Description covers edge cases and gotchas
□ Parameter names are self-documenting
□ Parameter descriptions include format requirements
□ Input is validated (poka-yoke — catch errors early)
□ Ambiguous inputs are eliminated (absolute paths, exact matches)
□ Model doesn't need to count lines, escape strings, or track positions
□ Output format is natural and includes context for reasoning
□ Tested with actual model usage and iterated on failures
```

## Next Steps

- [Prompt Chaining](../02-workflow-patterns/prompt-chaining.md) — compose well-designed tools into workflows
- [Routing](../02-workflow-patterns/routing.md) — direct tasks to the right specialized tools
- [Orchestrator-Workers](../02-workflow-patterns/orchestrator-workers.md) — dynamic tool selection at scale
