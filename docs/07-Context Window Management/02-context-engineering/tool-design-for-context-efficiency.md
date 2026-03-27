# Tool Design for Context Efficiency

> Building tools that return maximum information with minimum tokens.

---

## Why Tool Design Matters

In agentic workflows, **tool results dominate context consumption**. A single file read can inject 1,000+ tokens. A search result might add 5,000. Over a session with dozens of tool calls, tool results can consume 80%+ of the context window.

```
Typical agentic session context breakdown:
  System prompt:        3%
  Tool definitions:     5%
  Tool results:        65%  ← The biggest consumer
  Conversation:        20%
  Current query:        7%
```

---

## Principles of Token-Efficient Tool Design

### 1. Return Only What's Needed
```
❌ Bad: read_file("package.json") → returns entire 500-line file

✅ Good: read_json_field("package.json", "scripts") → returns only scripts section
```

### 2. Summarize Large Outputs
```
❌ Bad: search returns 50 full file contents

✅ Good: search returns file paths + line numbers + snippets
```

### 3. Support Pagination
```
❌ Bad: list_directory("/src") → returns 500 files recursively

✅ Good: list_directory("/src", depth=1) → returns top-level only
```

### 4. Structured Output
```
❌ Bad: "The test passed. All 247 tests ran successfully.
    Coverage was 84.2%. No failures detected..."

✅ Good: { "passed": 247, "failed": 0, "coverage": 84.2 }
```

---

## Tool Result Management

### Clear Old Results
Once a tool result has been used and its information absorbed into the conversation, clear it:

```
Turn 1: Read file → 1,500 tokens of content
Turn 2: AI analyzes and responds
Turn 3: Clear the file content from Turn 1 (replaced with summary)
```

This is called **tool result clearing** — one of the simplest forms of context management.

### Compress Verbose Outputs
```
Build logs:  10,000 tokens → "Build succeeded (47s, 0 warnings)"
Test output:  5,000 tokens → "All 312 tests passed (coverage: 87%)"
npm install:  3,000 tokens → "Installed 847 packages (0 vulnerabilities)"
```

---

## Tool Definition Optimization

Tool schemas themselves consume tokens. Optimize them:

```json
// ❌ Verbose tool definition (~200 tokens)
{
  "name": "read_file_from_filesystem",
  "description": "This tool reads the contents of a file from the filesystem. It takes a path parameter which should be the absolute path to the file you want to read. The file must exist and be readable.",
  "parameters": {
    "path": {
      "type": "string",
      "description": "The absolute filesystem path to the file that should be read"
    }
  }
}

// ✅ Concise tool definition (~100 tokens)
{
  "name": "read_file",
  "description": "Read file contents. Path must be absolute.",
  "parameters": {
    "path": { "type": "string", "description": "Absolute file path" }
  }
}
```

### Minimal Viable Tool Set
Don't provide 50 tools when 10 will do:
- Each tool definition costs tokens every turn
- Overlapping tools confuse the model
- If a human can't tell which tool to use, neither can the model

---

## Key Takeaways

1. **Tool results dominate context** — 65%+ of typical agentic sessions
2. **Return summaries, not dumps** — "247 passed" not full test output
3. **Clear old results** — Tool outputs from 10 turns ago are rarely needed
4. **Concise tool definitions** — Every tool schema costs tokens each turn
5. **Fewer, better tools** — 10 clear tools > 50 overlapping tools

---

## Next Steps

- 🔗 [Token Counting & Budgeting](../03-token-management/token-counting-and-budgeting.md) — Measure and plan
- 🔗 [Just-in-Time Retrieval](../07-context-retrieval/just-in-time-retrieval.md) — Load on demand
