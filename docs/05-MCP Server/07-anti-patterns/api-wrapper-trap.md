# The API Wrapper Trap

> Why mapping your REST API 1:1 to MCP tools is the #1 architectural mistake.

---

## The Anti-Pattern

```
REST API:
  POST   /issues          → create_issue
  GET    /issues           → list_issues
  GET    /issues/:id       → get_issue
  PATCH  /issues/:id       → update_issue
  DELETE /issues/:id       → delete_issue
  POST   /issues/:id/labels → add_label
  POST   /issues/:id/assign → assign_user
  POST   /issues/:id/comment → add_comment

"Let's just make each endpoint a tool!"
→ 8 tools for one feature
→ AI must orchestrate them in the right order
→ Context window wasted on definitions
→ Tool selection accuracy drops
```

---

## Why It Fails

### 1. LLMs Are Not API Clients

REST APIs are designed for programmatic clients that know the exact sequence of calls. LLMs are conversational agents that think in terms of **tasks**, not **HTTP methods**.

### 2. Orchestration Burden

When you expose low-level CRUD operations, the AI must:
- Choose the right sequence of tools
- Pass data between calls correctly
- Handle partial failures mid-sequence
- Manage state across multiple calls

This is brittle and error-prone.

### 3. Context Waste

Each tool definition costs ~200-500 tokens. 8 CRUD tools = ~3,000 tokens of definitions for one feature. That's context window the AI can't use for reasoning.

---

## The Solution: Task-Oriented Tools

Instead of mirroring your API, design tools around **what users want to accomplish**:

```
User tasks:
  "File a bug report"     → file_bug_report (creates + labels + assigns + notifies)
  "Check project status"  → get_project_summary (aggregates issues + PRs + metrics)
  "Review and merge PR"   → review_and_merge (gets diff + posts review + merges)
```

### Example Transformation

```python
# ❌ API Wrapper: 4 separate tools
@mcp.tool
async def create_issue(owner, repo, title, body): ...
@mcp.tool
async def add_labels(owner, repo, issue_number, labels): ...
@mcp.tool
async def assign_issue(owner, repo, issue_number, assignees): ...
@mcp.tool
async def add_comment(owner, repo, issue_number, comment): ...

# ✅ Task-Oriented: 1 tool
@mcp.tool
async def file_bug_report(
    owner: str, repo: str, title: str, description: str,
    severity: str = "medium", assignee: str = None
) -> str:
    """File a complete bug report. Creates an issue with appropriate labels,
    assigns it to the right person, and adds a structured description.
    Severity: low, medium, high, critical."""
    
    # Create issue with structured template
    issue = await github.create_issue(owner, repo, title, 
        body=format_bug_template(description, severity))
    
    # Add labels based on severity
    labels = ["bug", f"priority:{severity}"]
    await github.add_labels(owner, repo, issue.number, labels)
    
    # Assign if specified
    if assignee:
        await github.assign(owner, repo, issue.number, [assignee])
    
    return f"Bug report filed: #{issue.number} - {title}\nLabels: {', '.join(labels)}"
```

---

## When CRUD Tools Are OK

Not every case requires task-oriented tools. CRUD is acceptable when:

- The domain is simple (single-entity operations)
- Users genuinely need fine-grained control
- The operations are truly independent
- You have very few entities

---

## Key Takeaways

1. **Don't mirror your REST API** — Design for user tasks, not HTTP methods
2. **One tool per task** — Combine related operations into single tools
3. **Less is more** — 5 task-oriented tools beats 25 CRUD endpoints
4. **The AI isn't an API client** — It thinks in tasks, not request sequences
5. **Save context** — Fewer, richer tool definitions
