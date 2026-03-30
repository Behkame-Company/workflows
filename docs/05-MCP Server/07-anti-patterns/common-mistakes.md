# Common MCP Mistakes — Top 10

> The most frequent pitfalls when building or configuring MCP servers, with fixes for each.

---

## 1. Too Many Tools (The "Kitchen Sink" Server)

**Symptom**: AI picks the wrong tool or gets confused between similar tools.

```
❌ Server with 50+ tools covering GitHub, Jira, Slack, DB, files, email...
✅ 3-5 focused servers: github-server, jira-server, database-server
```

**Fix**: Keep each server to 10-15 tools. Split by domain into multiple servers.

---

## 2. Vague Tool Descriptions

**Symptom**: AI never selects the right tool or uses tools in wrong contexts.

```
❌ "Handles issues"
✅ "Search for GitHub issues by query, label, or assignee. Returns title, number, and status. Use when the user asks about existing bugs or feature requests."
```

**Fix**: Descriptions should answer: What? When? What does it return?

---

## 3. Printing to stdout (stdio transport)

**Symptom**: Client can't parse server responses; connection drops.

```python
❌ print("Processing request...")      # Corrupts the stdio stream!
✅ print("Processing...", file=sys.stderr)  # Safe — goes to logs
```

**Fix**: ALL logging must go to stderr. stdout is exclusively for MCP messages.

---

## 4. Raw Data Dumps

**Symptom**: AI responses become confused or truncated; high token costs.

```
❌ Returning entire 500-row database query as raw JSON
✅ Returning summarized results with pagination: "Found 500 rows. Top 20 shown."
```

**Fix**: Summarize, paginate, and truncate all responses.

---

## 5. Unhelpful Error Messages

**Symptom**: AI can't recover from errors; tells user "something went wrong."

```
❌ "Error: 404"
✅ "Error: Repository 'myorg/wepapp' not found. Did you mean 'myorg/webapp'? Use list_repos to see available repositories."
```

**Fix**: Include what happened, why, and what to do next.

---

## 6. Deep Nested Schemas

**Symptom**: AI passes wrong structure, misses required fields.

```json
❌ { "config": { "output": { "format": "json" } } }
✅ { "output_format": "json" }
```

**Fix**: Flatten to ≤6 top-level parameters.

---

## 7. No Input Validation

**Symptom**: Server crashes or produces garbage output on bad input.

```python
❌ Just passing raw input to database query
✅ Validating SQL starts with SELECT, checking for injection patterns
```

**Fix**: Validate every input parameter before processing.

---

## 8. Secrets in Configuration

**Symptom**: API keys committed to git or visible in logs.

```json
❌ "args": ["--api-key", "sk-abc123def456"]
✅ "env": { "API_KEY": "${SECRET_API_KEY}" }
```

**Fix**: Use environment variables. Never hard-code secrets.

---

## 9. Ignoring Capability Negotiation

**Symptom**: Server tries to use features the client doesn't support.

```
❌ Server sends sampling request to a client that doesn't support sampling
✅ Server checks client capabilities before using advanced features
```

**Fix**: Always check `client.capabilities` before using optional features.

---

## 10. Over-Engineering the Server

**Symptom**: Server is slow, complex, and tries to do the AI's job.

```
❌ Server analyzes errors, generates summaries, makes recommendations
✅ Server returns raw data; AI does the analysis
```

**Fix**: Servers provide data and actions. The LLM does the thinking.

---

## Quick Reference

| Mistake | Impact | Fix |
|---------|--------|-----|
| Too many tools | Wrong tool selection | Split into focused servers |
| Vague descriptions | AI guesses wrong | What + When + Returns |
| stdout pollution | Connection breaks | Use stderr for logs |
| Raw data dumps | Context overflow | Summarize + paginate |
| Unhelpful errors | No recovery | Prescriptive error messages |
| Deep nesting | Schema mismatches | Flatten parameters |
| No validation | Crashes/exploits | Validate all inputs |
| Secrets in config | Security breach | Environment variables |
| Ignoring capabilities | Feature failures | Check before using |
| Over-engineering | Slow + complex | Let the LLM reason |
