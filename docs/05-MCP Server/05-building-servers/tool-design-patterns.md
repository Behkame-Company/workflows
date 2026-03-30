# Tool Design Patterns

> Design tools that AI agents actually use correctly — patterns from production MCP servers.

---

## The Golden Rules

1. **Design for tasks, not APIs** — What does the user want to accomplish?
2. **Describe for discovery** — The AI reads descriptions to choose tools
3. **Simplify for success** — Flat schemas, enums, sensible defaults
4. **Fail for recovery** — Error messages that help the AI retry

---

## Pattern 1: Task-Oriented Tools

```
❌ API-Oriented (mirrors REST endpoints):
  create_issue, add_label, assign_user, add_comment, set_milestone
  → 5 tools the AI must orchestrate correctly

✅ Task-Oriented (mirrors user intent):
  file_bug_report → creates issue + labels "bug" + assigns on-call + adds template
  → 1 tool that does the right thing
```

### When to Combine

Combine when:
- Actions are always done together
- The user thinks of them as one task
- Splitting them creates error-prone multi-step sequences

Keep separate when:
- Actions are genuinely independent
- Different users need different combinations
- The tool would have too many parameters (>6)

---

## Pattern 2: Progressive Disclosure

Start with high-level tools, provide drill-down tools for details:

```
Level 1: search_issues       → "Find issues matching a query"
Level 2: get_issue_details    → "Get full details for a specific issue"
Level 3: get_issue_comments   → "Get comments on a specific issue"
```

The AI naturally starts broad and narrows down.

---

## Pattern 3: Smart Defaults

Reduce required parameters by providing intelligent defaults:

```json
{
  "name": "create_branch",
  "inputSchema": {
    "properties": {
      "name": { "type": "string", "description": "Branch name" },
      "base": {
        "type": "string",
        "description": "Base branch. Defaults to 'main' if not specified."
      }
    },
    "required": ["name"]  // Only name is required; base defaults to main
  }
}
```

---

## Pattern 4: Confirmation for Destructive Actions

Tools that delete, deploy, or modify critical data should include a confirmation parameter:

```json
{
  "name": "delete_branch",
  "description": "Delete a Git branch. Requires explicit confirmation.",
  "inputSchema": {
    "properties": {
      "branch": { "type": "string" },
      "confirm": {
        "type": "boolean",
        "description": "Must be true to proceed with deletion"
      }
    },
    "required": ["branch", "confirm"]
  }
}
```

---

## Pattern 5: Batch Operations

When users often need to process multiple items:

```json
{
  "name": "add_labels",
  "description": "Add one or more labels to an issue.",
  "inputSchema": {
    "properties": {
      "issue_number": { "type": "integer" },
      "labels": {
        "type": "array",
        "items": { "type": "string" },
        "description": "Labels to add (e.g., ['bug', 'priority:high'])"
      }
    }
  }
}
```

---

## Pattern 6: Contextual Error Messages

```python
@mcp.tool
async def query_database(sql: str) -> str:
    """Run a read-only SQL query against the application database."""
    try:
        if sql.strip().upper().startswith(("INSERT", "UPDATE", "DELETE", "DROP")):
            return "Error: Only SELECT queries are allowed. This tool is read-only. Use 'execute_migration' for write operations."
        results = await db.execute(sql)
        return format_results(results)
    except SyntaxError:
        return f"Error: Invalid SQL syntax. Check your query: {sql}"
    except TimeoutError:
        return "Error: Query timed out after 30s. Try adding LIMIT or filtering with WHERE clause."
```

Key: Tell the AI **what went wrong** and **what to do instead**.

---

## Pattern 7: Paginated Results

For tools that can return large datasets:

```json
{
  "name": "search_code",
  "inputSchema": {
    "properties": {
      "query": { "type": "string" },
      "page": { "type": "integer", "description": "Page number (starts at 1)" },
      "per_page": { "type": "integer", "description": "Results per page (max 50)" }
    }
  }
}
```

Return pagination metadata in results:
```
Found 142 results (showing 1-20). Use page=2 to see more.
```

---

## Anti-Patterns to Avoid

For naming conventions and schema design principles, see [Tool Naming & Schemas](../06-best-practices/tool-naming-and-schemas.md).

| Anti-Pattern | Problem | Fix |
|-------------|---------|-----|
| **Too many tools** | AI can't choose correctly with 50+ tools | Keep under 10-15; use multiple servers |
| **No error context** | `"Error: 400"` | `"Error: Repository not found. Check owner/repo format."` |
| **Raw data dumps** | Returning 10,000-line JSON | Summarize, paginate, or filter |

---

## Key Takeaways

1. **Task-oriented** — One tool per user task, not per API endpoint
2. **Smart defaults** — Minimize required parameters
3. **Contextual errors** — Tell the AI what went wrong and what to try
4. **Progressive disclosure** — Broad search → specific details
5. **Under 10-15 tools** per server — keep focused
