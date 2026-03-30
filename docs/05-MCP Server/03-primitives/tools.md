# Tools — MCP's Executable Functions

> Tools are the "verbs" of MCP — actions the AI agent can invoke to interact with the real world.

---

## What Are Tools?

Tools are **executable functions** exposed by an MCP server that AI agents can call. They represent actions: creating issues, querying databases, sending notifications, running tests.

```
AI Agent: "I need to create a GitHub issue"
  → discovers tool: create_issue
  → calls tool with: { owner: "myorg", repo: "app", title: "Fix bug" }
  → receives result: "Created issue #247"
```

---

## Tool Definition Schema

Every tool is defined with a name, description, and input schema:

```json
{
  "name": "create_issue",
  "description": "Create a new issue in a GitHub repository. Use this when the user wants to report a bug, request a feature, or track a task.",
  "inputSchema": {
    "type": "object",
    "properties": {
      "owner": {
        "type": "string",
        "description": "The GitHub organization or username that owns the repo"
      },
      "repo": {
        "type": "string",
        "description": "The repository name (not the full URL)"
      },
      "title": {
        "type": "string",
        "description": "A concise, descriptive issue title"
      },
      "body": {
        "type": "string",
        "description": "Detailed issue description in markdown"
      },
      "labels": {
        "type": "array",
        "items": { "type": "string" },
        "description": "Labels to apply (e.g., 'bug', 'enhancement')"
      }
    },
    "required": ["owner", "repo", "title"]
  }
}
```

---

## Tool Lifecycle

Tools are discovered during the **Discovery** phase (via `tools/list`) and invoked during the **Operation** phase (via `tools/call`). See [Protocol Lifecycle](../02-architecture/protocol-lifecycle.md) for phase details.

The basic flow: **Discover → Select → Invoke → Execute → Respond**.

---

## Tool Design Best Practices

### 1. Task-Oriented, Not API-Oriented

```
❌ Bad: Mirror every REST endpoint as a tool
  - create_issue, update_issue, add_label, assign_user, add_comment (5 tools)

✅ Good: Design for user tasks
  - file_bug_report (creates issue + adds labels + assigns + comments) (1 tool)
```

### 2. Clear, Specific Descriptions

The description is the AI's **only guide** for choosing the right tool:

```
❌ "Creates an issue"
✅ "Create a new issue in a GitHub repository. Use this when the user wants to 
    report a bug, request a feature, or track a task. Requires owner, repo, 
    and title. Optionally accepts body, labels, and assignees."
```

### 3. Simple, Flat Schemas

LLMs handle flat parameters better than deeply nested objects:

```
❌ Deep nesting:
  { "config": { "output": { "format": { "type": "json" } } } }

✅ Flat parameters:
  { "output_format": "json" }
```

### 4. Use Enums Over Free-Form Strings

```
❌ "priority": { "type": "string" }
✅ "priority": { "type": "string", "enum": ["low", "medium", "high", "critical"] }
```

### 5. Limit to 6 Top-Level Parameters

Research shows LLMs perform best with ≤6 parameters per tool. Bundle related fields into logical groups if you need more.

---

## Tool Response Patterns

### Text Response (Most Common)
```json
{
  "content": [{ "type": "text", "text": "Operation completed successfully" }],
  "isError": false
}
```

### Structured Data Response
```json
{
  "content": [{
    "type": "text",
    "text": "Found 3 issues:\n1. #241 - Login timeout\n2. #243 - CSS broken\n3. #247 - 500 error"
  }],
  "isError": false
}
```

### Error Response
```json
{
  "content": [{
    "type": "text",
    "text": "Failed to create issue: Repository 'myorg/webapp' not found. Check that the repository exists and your token has access."
  }],
  "isError": true
}
```

💡 **Pro tip**: Error messages should tell the AI **why** it failed and **what to try next**.

---

## Key Takeaways

1. Tools are the most-used MCP primitive — they let AI agents **do things**
2. Design tools for **tasks**, not API endpoints
3. Descriptions are critical — they're how the AI decides which tool to use
4. Keep schemas **flat** and parameters **under 6**
5. Error messages should be **actionable** for the AI
