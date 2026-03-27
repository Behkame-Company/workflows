# Performance & Context Budget

> Managing tokens, latency, and payload sizes for efficient MCP interactions.

---

## The Context Window Problem

Every tool definition, resource, and response competes for space in the AI's context window. An overloaded context means:
- The AI forgets earlier instructions
- Tool selection accuracy drops
- Response quality degrades
- Token costs increase

---

## Budget Guidelines

### Tool Definitions
| Metric | Guideline |
|--------|-----------|
| **Tools per server** | 10-15 maximum |
| **Description length** | 1-3 sentences (50-150 words) |
| **Parameters per tool** | ≤6 top-level |
| **Total tool definitions** | Keep under 4KB across all connected servers |

### Tool Responses
| Metric | Guideline |
|--------|-----------|
| **Response size** | Under 5,000 characters for most tools |
| **Large datasets** | Paginate at 20-50 items per page |
| **File contents** | Truncate at 10,000 characters with note |
| **Error messages** | Under 500 characters |

### Resources
| Metric | Guideline |
|--------|-----------|
| **Resource size** | Under 10,000 characters |
| **Summary resources** | Prefer summaries over raw dumps |
| **Dynamic filtering** | Let the AI request specific subsets |

---

## Optimization Techniques

### 1. Lazy Loading
Don't load everything at discovery — load details on demand:
```
tools/list → Returns names + descriptions (lightweight)
tools/call → Executes and returns full results (on demand)
```

### 2. Pagination
```python
@mcp.tool
async def search_issues(query: str, page: int = 1, per_page: int = 20) -> str:
    results = await github.search(query, page=page, per_page=per_page)
    total = results.total_count
    items = format_issues(results.items)
    return f"{items}\n\n--- Page {page} of {(total + per_page - 1) // per_page} ({total} total results) ---"
```

### 3. Summarization
```python
@mcp.tool
async def get_repo_overview(owner: str, repo: str) -> str:
    """Get a high-level overview of a repository."""
    repo = await github.get_repo(owner, repo)
    return f"""Repository: {repo.full_name}
Description: {repo.description}
Language: {repo.language}
Stars: {repo.stargazers_count}
Open Issues: {repo.open_issues_count}
Last Updated: {repo.updated_at}
Default Branch: {repo.default_branch}"""
    # NOT: return json.dumps(repo.raw_data)  ← would be 50KB+
```

### 4. Streaming for Long Operations
Use progress notifications instead of blocking:
```python
@mcp.tool
async def run_tests(suite: str) -> str:
    """Run test suite and report results."""
    await mcp.notify_progress("Running tests...", progress=0, total=100)
    results = await test_runner.run(suite)
    await mcp.notify_progress("Tests complete", progress=100, total=100)
    return f"Results: {results.passed} passed, {results.failed} failed"
```

---

## Latency Considerations

| Transport | Typical Latency | Optimization |
|-----------|----------------|-------------|
| **stdio** | <1ms | Already optimal |
| **HTTP (local)** | 1-5ms | Keep alive connections |
| **HTTP (remote)** | 50-200ms | Batch related operations, cache results |
| **HTTP (with auth)** | 100-500ms | Cache tokens, minimize auth handshakes |

---

## Key Takeaways

1. **Context is finite** — Every byte of tool definitions and responses costs tokens
2. **Summarize, don't dump** — Return what the AI needs, not everything you have
3. **Paginate large results** — 20-50 items per page is the sweet spot
4. **Keep tool count low** — 10-15 per server; use multiple servers for more
5. **Stream long operations** — Progress notifications over blocking responses

---

## Next Steps

- 🔗 [Common Mistakes](../07-anti-patterns/common-mistakes.md) — Performance anti-patterns
- 🔗 [Enterprise Deployment](../10-advanced/enterprise-deployment.md) — Scaling considerations
