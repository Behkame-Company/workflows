# Over-Engineering MCP Servers

> When your MCP server tries to be too smart, too complex, or does the AI's job.

---

## The Anti-Pattern

```
"Let's make the server intelligent!"
  → Server analyzes data before returning it
  → Server makes recommendations
  → Server caches user preferences
  → Server manages complex state machines
  → Server becomes a mini-application

Result: Slow, brittle, hard to maintain, duplicates LLM reasoning
```

---

## Signs of Over-Engineering

### 1. The Server Does Analysis
```
❌ Server returns: "Based on analysis of 500 errors, the root cause is a 
   connection pool issue. Recommended fix: increase pool size to 20."

✅ Server returns: "Last 500 errors (summary):
   - ConnectionPool exhausted: 340 occurrences
   - Query timeout: 120 occurrences  
   - Auth failure: 40 occurrences"
```

The LLM should do the analysis. The server should provide the data.

### 2. Complex Internal State
```
❌ Server tracks: user session, conversation history, previous queries,
   user preferences, undo stack, transaction log...

✅ Server: stateless. Each tool call is self-contained.
   Client passes all needed context in the request.
```

### 3. Too Much Business Logic
```
❌ Server enforces: "Don't create issues on weekends" / "Require manager 
   approval for production deploys" / "Auto-assign based on team rotation"

✅ Server: creates issues when asked, deploys when asked.
   Business rules live in your application, not the MCP server.
```

### 4. Custom Orchestration Engine
```
❌ Server has its own workflow engine that chains tool calls together

✅ Server exposes simple tools. The AI (or a proper orchestrator) 
   chains them as needed.
```

---

## The Right Level of Complexity

| Do in the Server | Don't Do in the Server |
|---|---|
| Input validation | Business logic |
| Data formatting | Data analysis |
| Error handling | Error root-cause diagnosis |
| Pagination | Content summarization |
| Authentication | User preference management |
| Simple transformations | Complex orchestration |
| Rate limiting | State machines |

---

## Refactoring Checklist

If your server is over-engineered, simplify:

1. **Remove analysis logic** — Return raw (but formatted) data
2. **Eliminate state** — Make every tool call self-contained
3. **Move business rules** — Put them in your application, not the MCP server
4. **Split complex tools** — If a tool does 5 things, split into focused tools
5. **Delete caching** — Unless it's pure performance optimization

---

## Key Takeaways

1. **Servers provide data and actions** — the LLM does the thinking
2. **Stateless is better** — each call should be independent
3. **Simple tools, smart AI** — keep the server focused and composable
4. **Business logic lives elsewhere** — not in the MCP server
5. **If you're writing "if/else analysis" in a tool, you're doing it wrong**

---

## Next Steps

- 🔗 [Server Design Principles](../06-best-practices/server-design-principles.md) — The right approach
- 🔗 [Security Model](../08-security/security-model.md) — Where complexity IS needed
