# Tool-Augmented Prompting

> Prompting models to use tools effectively — from search to code execution.

---

## Why Tools Matter

Without tools, models can only reason from their training data. With tools, they can:
- Read actual files (not guess at contents)
- Run tests (not assume they pass)
- Search codebases (not hallucinate file names)
- Execute code (not predict output)

---

## Tool Use Best Practices

### 1. Be Explicit About When to Use Tools
```
When investigating a bug:
- ALWAYS read the actual file before suggesting changes
- ALWAYS run existing tests before and after your changes
- Use grep to find all usages of a function before renaming it
- Use glob to verify a file exists before trying to read it
```

### 2. Tool Selection Guidance
```
Choose the right tool:
- Finding files by name → glob
- Finding text in files → grep
- Understanding a function → read the file
- Verifying code works → run tests
- Checking project setup → read package.json / tsconfig.json
```

### 3. Reduce Overtriggering (Claude 4.x)
Modern Claude models are very proactive about tool use. Dial back if needed:
```
Use tools when they genuinely help. Don't:
- Spawn subagents for tasks you can do with a simple grep
- Read every file in a directory when you only need one
- Run tests after cosmetic changes (comments, whitespace)

Do use tools when:
- You need to verify file contents (don't assume)
- You need to check test results
- You need to find something in the codebase
```

### 4. Efficient Tool Usage
```
Maximize efficiency:
- Batch related operations: read multiple files at once
- Chain commands: npm run build && npm run test
- Use grep with glob patterns to narrow search scope
- Suppress verbose output: use --quiet, pipe to head
```

---

## Tool Descriptions in Prompts

When defining tools for the model, descriptions matter:

### ❌ Vague Tool Description
```
"search" - searches for things
```

### ✅ Clear Tool Description
```
"search_codebase" - Search for text patterns in source files.
  Input: { pattern: string, path?: string, fileType?: string }
  Use when: You need to find where a function/variable/pattern is used.
  Do NOT use for: Finding files by name (use list_files instead).
```

**Good tool descriptions include**: what it does, when to use it, when NOT to use it, and input format.

---

## Tool Results as Context

Tool results often dominate the context window (65%+ of total tokens). Optimize:

```
When reading files:
- Read specific functions, not entire files when possible
- Use line ranges when you know where the relevant code is
- After reading a large file, summarize what you found 
  before moving to the next file

When searching:
- Start with narrow searches (specific function name)
- Only broaden if narrow search returns nothing
- Limit results: head -20 for large result sets
```

---

## Parallel Tool Execution

Modern models excel at parallel tool use:

```
To build context for a code change, read these files simultaneously:
1. The source file being modified
2. Its test file
3. The types/interfaces file it imports from
4. The configuration it depends on

Don't read them sequentially — batch the reads.
```

Anthropic notes Claude can sometimes "bottleneck system performance" with parallel bash commands — this is usually a good problem to have.

---

## Next Steps

- 🔗 [Agentic Prompting](agentic-prompting.md) — Autonomous agent patterns
- 🔗 [ReAct Pattern](react-pattern.md) — Reasoning + Acting framework
