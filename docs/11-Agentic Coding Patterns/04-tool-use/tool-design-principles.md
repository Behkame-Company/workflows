# Tool Design Principles

> Invest as much effort in your Agent-Computer Interface (ACI) as you would in a Human-Computer Interface (HCI) — great tool design is the single biggest lever for agent performance.

---

## The Core Insight

From Anthropic's research on building effective agents:

> *"Just as a poor UI can make an application unusable for humans, a poor ACI can make a tool unusable for agents."*

Tool design is not an afterthought. The way you describe, structure, and constrain your tools directly determines whether an agent succeeds or fails. SWE-bench results showed that **switching from relative to absolute filepaths** in a single tool eliminated an entire class of errors — no model changes needed.

## The Five Principles

```
┌──────────────────────────────────────────────────────────────────┐
│                   TOOL DESIGN PRINCIPLES                          │
│                                                                  │
│  1. CLEAR DESCRIPTIONS    Write great docstrings                 │
│  2. NATURAL FORMATS       Match what models expect               │
│  3. GIVE TOKENS TO THINK  Don't force models into corners        │
│  4. POKA-YOKE             Make mistakes harder                   │
│  5. TEST AND ITERATE      Run examples, see failures, improve    │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Principle 1: Clear Descriptions

Write tool descriptions like you're writing a **great docstring for a junior developer** — include examples, edge cases, and format requirements.

```typescript
// ❌ BAD: Vague description
const searchTool = {
  name: "search",
  description: "Search for code"
};

// ✅ GOOD: Detailed, example-rich description
const searchTool = {
  name: "search_codebase",
  description: `Search for text patterns in the codebase using regex.
    
    Returns matching lines with file paths and line numbers.
    
    Parameters:
    - pattern: Regex pattern to search for (e.g., "function handleAuth")
    - path: Directory to search in. Use absolute paths (e.g., "/workspace/src")
    - fileGlob: Optional file type filter (e.g., "*.ts", "*.{ts,tsx}")
    
    Examples:
    - Find all API routes: pattern="router\\.(get|post|put|delete)", path="/workspace/src"
    - Find TODO comments: pattern="TODO|FIXME", path="/workspace"
    - Find imports of a module: pattern="from ['\"]./auth", fileGlob="*.ts"
    
    Note: Returns max 50 results. Use a more specific pattern or path to narrow results.
    For exact string matching, escape regex special characters.`
};
```

### Principle 2: Natural Formats

Keep inputs and outputs close to what the model has seen in training data. Avoid formats that require counting, complex escaping, or unusual structures.

```typescript
// ❌ BAD: Line-number-based diff format (models must count lines)
const editTool_bad = {
  name: "edit",
  description: "Edit a file using line numbers",
  parameters: {
    startLine: "number — line to start editing",
    endLine: "number — line to stop editing",
    newContent: "string — replacement content"
  }
};
// Agent must count lines correctly — a common source of errors

// ✅ GOOD: Search-and-replace format (models match text naturally)
const editTool_good = {
  name: "edit",
  description: `Edit a file by replacing exact text matches.
    
    Parameters:
    - path: Absolute path to the file
    - oldText: Exact text to find (must match file content precisely)
    - newText: Text to replace it with
    
    The oldText must be unique in the file. Include enough surrounding
    context to ensure uniqueness.
    
    Example:
      path: "/workspace/src/auth.ts"
      oldText: "const token = jwt.sign(payload, SECRET)"
      newText: "const token = jwt.sign(payload, SECRET, { expiresIn: '1h' })"`,
};
```

```typescript
// ❌ BAD: JSON with escaped strings inside strings
const output_bad = {
  result: "{\"files\": [\"src\\\\auth.ts\", \"src\\\\middleware\\\\jwt.ts\"]}"
};
// Model must parse escaped JSON inside JSON — error-prone

// ✅ GOOD: Native structured data
const output_good = {
  files: ["src/auth.ts", "src/middleware/jwt.ts"],
  matchCount: 2,
  searchPattern: "jwt.verify"
};
```

### Principle 3: Give Tokens to Think

Don't force the model to make all decisions in one step. When a tool requires complex input, break it into stages or provide a way for the model to reason incrementally.

```typescript
// ❌ BAD: Model must compose entire file in one shot
const writeFile = {
  name: "write_file",
  description: "Write complete file contents",
  parameters: { path: "string", content: "string — entire file content" }
};

// ✅ GOOD: Allow incremental edits
const editFile = {
  name: "edit_file",
  description: "Make a targeted edit to an existing file",
  parameters: { 
    path: "string", 
    oldText: "string — text to replace", 
    newText: "string — replacement text" 
  }
};
// Model can make multiple small edits, thinking between each one
```

### Principle 4: Poka-Yoke (Error-Proofing)

Design tools so that **mistakes are harder to make**. Constrain inputs to reduce the space of possible errors.

```
┌────────────────────────────────────────────────────────────────┐
│                   POKA-YOKE EXAMPLES                            │
│                                                                │
│  BEFORE                        AFTER                           │
│  ─────────────────────         ──────────────────────          │
│  Relative paths                Absolute paths                  │
│  "auth.ts"                     "/workspace/src/auth.ts"        │
│  → Agent guesses cwd           → Unambiguous location          │
│                                                                │
│  Free-form line numbers        Search-and-replace              │
│  "edit line 42-45"             "replace 'old' with 'new'"      │
│  → Off-by-one errors           → Text matching is natural      │
│                                                                │
│  Unbounded output              Paginated results               │
│  "return all matches"          "return first 50, has_more flag" │
│  → Context overflow            → Controllable output size      │
│                                                                │
│  Silent failures               Explicit error messages         │
│  "edit applied" (even if not)  "ERROR: oldText not found in    │
│                                 file. Similar text on line 42:" │
│  → Agent thinks it succeeded   → Agent can self-correct        │
└────────────────────────────────────────────────────────────────┘
```

```typescript
// Poka-yoke in practice: validate inputs and give helpful errors
async function editFile(params: EditParams): Promise<ToolResult> {
  // Poka-yoke: require absolute paths
  if (!path.isAbsolute(params.path)) {
    return {
      success: false,
      error: `Path must be absolute. Got: "${params.path}". 
              Did you mean "/workspace/${params.path}"?`
    };
  }

  // Poka-yoke: verify old text exists
  const content = await fs.readFile(params.path, "utf-8");
  if (!content.includes(params.oldText)) {
    const similar = findSimilarText(content, params.oldText);
    return {
      success: false,
      error: `oldText not found in ${params.path}. 
              ${similar ? `Similar text found on line ${similar.line}:\n"${similar.text}"` : 
                         "File may have been modified. Re-read it first."}`
    };
  }

  // Poka-yoke: check for multiple matches
  const matchCount = content.split(params.oldText).length - 1;
  if (matchCount > 1) {
    return {
      success: false,
      error: `oldText matches ${matchCount} locations. Include more surrounding 
              context to make it unique.`
    };
  }

  // Safe to proceed
  const newContent = content.replace(params.oldText, params.newText);
  await fs.writeFile(params.path, newContent);
  return { success: true, message: "Edit applied successfully" };
}
```

### Principle 5: Test and Iterate

Run many examples through your tools, observe where the agent makes mistakes, and iterate on the design.

```typescript
// Tool testing checklist
const toolTestCases = [
  // Happy path
  { input: { pattern: "handleAuth", path: "/workspace/src" }, expect: "finds matches" },
  
  // Edge cases
  { input: { pattern: "class {}", path: "/workspace" }, expect: "handles regex special chars" },
  { input: { pattern: ".*", path: "/workspace" }, expect: "handles broad patterns gracefully" },
  
  // Error cases
  { input: { pattern: "test", path: "src/" }, expect: "rejects relative path with helpful message" },
  { input: { pattern: "", path: "/workspace" }, expect: "rejects empty pattern" },
  
  // Agent confusion cases (found through testing)
  { input: { pattern: "import.*from", path: "/workspace" }, 
    expect: "returns paginated results, not 10000 lines" },
];
```

## Tool Boundary Clarity

When you have multiple tools, **clearly separate their responsibilities**. Ambiguous boundaries cause the agent to pick the wrong tool.

```typescript
// ❌ BAD: Overlapping tools
const tools_bad = [
  { name: "search", description: "Search for text in files" },
  { name: "find",   description: "Find files or text" },
  { name: "grep",   description: "Search using patterns" },
];
// Agent doesn't know which to use

// ✅ GOOD: Clear, distinct responsibilities
const tools_good = [
  { 
    name: "search_content", 
    description: "Search for text INSIDE files. Use when you know WHAT to find but not WHERE." 
  },
  { 
    name: "find_files", 
    description: "Find files by NAME or PATTERN. Use when you know the filename but not the path." 
  },
  { 
    name: "list_directory", 
    description: "List files IN a directory. Use to explore project structure." 
  },
];
```

## Client Tools vs. Server Tools

```
┌────────────────────────────────────────────────────────────────┐
│              TOOL EXECUTION MODELS                              │
│                                                                │
│  CLIENT TOOLS (you execute)     SERVER TOOLS (provider executes)│
│  ─────────────────────────      ──────────────────────────────  │
│                                                                │
│  ┌──────┐     ┌────────┐       ┌──────┐     ┌────────┐       │
│  │ LLM  │────►│ Your   │       │ LLM  │────►│API     │       │
│  │      │◄────│ Code   │       │      │◄────│Provider│       │
│  └──────┘     └────────┘       └──────┘     └────────┘       │
│                                                                │
│  • Shell commands               • Web search (via API)         │
│  • File read/write              • Code execution (sandboxed)   │
│  • Git operations               • Image generation             │
│  • MCP tool calls               • Retrieval services           │
│                                                                │
│  You control execution,         Provider controls execution,   │
│  security, and permissions      handles sandboxing             │
└────────────────────────────────────────────────────────────────┘
```

## Tool Design Checklist

Use this checklist when designing or reviewing tools for coding agents:

| Criterion | Question | Fix |
|-----------|----------|-----|
| **Description** | Would a junior dev know how to use this tool from the description alone? | Add examples and edge cases |
| **Format** | Are inputs/outputs close to natural text? | Avoid counting, escaping, complex encodings |
| **Error messages** | Does the tool explain what went wrong and suggest a fix? | Add context to error responses |
| **Boundaries** | Could the agent confuse this tool with another? | Clarify when to use each tool |
| **Constraints** | What mistakes are possible? Can any be prevented? | Add input validation, use absolute paths |
| **Output size** | Can the tool flood the context window? | Add pagination, truncation |
| **Idempotency** | Is calling the tool twice safe? | Use search-and-replace, not append |
