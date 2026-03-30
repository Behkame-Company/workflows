# Server Design Principles

> The core rules for building MCP servers that are reliable, composable, and AI-friendly.

---

## The 7 Principles

### 1. Single Responsibility

Each server should focus on **one domain**:

```
✅ github-server    → GitHub operations
✅ postgres-server  → Database queries  
✅ sentry-server    → Error tracking

❌ everything-server → GitHub + DB + Sentry + Files + Email + ...
```

**Why**: Focused servers are easier to maintain, test, and compose. The host combines multiple servers naturally.

### 2. Servers Are Data Providers, Not Reasoners

The LLM does the thinking. Your server provides the data and actions:

```
❌ Server analyzes errors and suggests fixes → duplicates LLM's job
✅ Server returns raw error data → LLM analyzes and suggests fixes
```

**Exception**: Simple transformations (formatting, filtering) are fine. Complex analysis belongs to the LLM.

### 3. Design for Discovery

The AI discovers your server's capabilities dynamically. Make it easy:

- **Tool descriptions** are the #1 factor in correct tool selection
- **Resource URIs** should be self-explanatory
- **Prompt names** should describe the workflow, not the implementation

### 4. Fail Gracefully

Never crash. Always return useful error messages:

```python
# Every tool should handle errors
@mcp.tool
async def query(sql: str) -> str:
    try:
        return await execute_query(sql)
    except ConnectionError:
        return "Error: Cannot connect to database. Check that the database is running and accessible."
    except TimeoutError:
        return "Error: Query timed out after 30s. Try simplifying the query or adding LIMIT."
    except Exception as e:
        return f"Error: Unexpected failure: {str(e)}. Please report this issue."
```

### 5. Right-Size Your Output

The AI's context window is limited. Don't waste it:

```python
# Truncate large results
if len(result) > 5000:
    return result[:5000] + f"\n\n[Truncated — {len(result)} total chars. Use pagination for more.]"

# Summarize when appropriate
return f"Found {len(issues)} issues. Top 5 by priority:\n{format_top_5(issues)}"
```

### 6. Be Stateless Where Possible

Each tool call should be independent. Don't rely on previous calls:

```
❌ Step 1: set_repo("myorg/webapp")
   Step 2: create_issue(title="Bug")  ← requires step 1 to have run

✅ create_issue(owner="myorg", repo="webapp", title="Bug")  ← self-contained
```

### 7. Respect Security Boundaries

- Never expose more than necessary
- Validate all inputs
- Use the least privilege possible
- Support human consent for destructive operations

---

## Key Takeaways

| Principle | One-Line Summary |
|-----------|-----------------|
| Single responsibility | One server, one domain |
| Data, not reasoning | Return data, let the LLM think |
| Design for discovery | Great descriptions = correct tool selection |
| Fail gracefully | Error messages that help the AI recover |
| Right-size output | Don't waste the context window |
| Be stateless | Each call is self-contained |
| Respect security | Validate, limit, and require consent |
