# Error Handling Best Practices

> Design error responses that help AI agents recover, retry, and provide useful feedback to users.

---

## The Error Handling Principle

**Errors should be prescriptive, not descriptive.** Tell the AI what to DO, not just what went wrong.

```
❌ "Error: 404 Not Found"
✅ "Error: Repository 'myorg/webapp' not found. Verify the owner and repo name are correct, or use search_repos to find available repositories."
```

---

## Error Response Patterns

### Tool-Level Errors (Use `isError: true`)
```json
{
  "content": [{
    "type": "text",
    "text": "Error: Cannot connect to database at localhost:5432. The database might be down. Try: 1) Check if PostgreSQL is running 2) Verify connection string 3) Check firewall rules."
  }],
  "isError": true
}
```

### Validation Errors (Prevent Bad Calls)
```python
@mcp.tool
async def query_database(sql: str, limit: int = 100) -> str:
    """Run a read-only SQL query."""
    # Validate before executing
    if not sql.strip():
        return "Error: SQL query cannot be empty. Provide a valid SELECT statement."
    
    if not sql.strip().upper().startswith("SELECT"):
        return "Error: Only SELECT queries allowed. For write operations, use execute_migration tool instead."
    
    if limit > 1000:
        return "Error: Maximum limit is 1000 rows. Reduce your limit or add WHERE clauses to narrow results."
    
    # Execute...
```

### Partial Success
```python
return f"""Completed with warnings:
- Created issue #247 ✅
- Added labels 'bug', 'priority:high' ✅  
- Failed to assign user 'jdoe': User not found in this repository ⚠️
  Suggestion: Use list_collaborators to find valid assignees."""
```

---

## Error Message Template

```
Error: [What happened].
[Why it happened (if known)].
[What to do about it].
```

### Examples by Category

| Category | Example |
|----------|---------|
| **Not found** | `"Error: Issue #999 not found in myorg/webapp. Check the issue number or use search_issues to find it."` |
| **Permission** | `"Error: Token lacks 'repo' scope. The current token can only read public repos. Ask the user to update their GitHub token."` |
| **Timeout** | `"Error: Query timed out after 30s. Try: 1) Add LIMIT 100, 2) Add WHERE clause, 3) Simplify JOINs."` |
| **Rate limit** | `"Error: GitHub API rate limit exceeded. Resets at 10:45 UTC. Wait 5 minutes and retry."` |
| **Invalid input** | `"Error: Date format must be ISO 8601 (e.g., '2026-03-26'). Received: 'March 26'."` |
| **Server error** | `"Error: Internal server error. This is a bug in the MCP server, not your request. Details logged for debugging."` |

---

## Key Takeaways

1. **Prescriptive errors** — Tell the AI what to do next, not just what failed
2. **Include suggestions** — Alternative tools, corrected formats, retry strategies
3. **Use `isError: true`** — So the AI knows this is an error, not normal output
4. **Validate early** — Catch bad inputs before making expensive calls
5. **Handle partial success** — Report what worked and what didn't

---

## Next Steps

- 🔗 [Performance & Context Budget](performance-and-context.md) — Managing payload sizes
- 🔗 [Common Mistakes](../07-anti-patterns/common-mistakes.md) — Errors in error handling
