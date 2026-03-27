# Tool Naming & Schema Best Practices

> How to name tools and structure schemas so AI agents select and invoke them correctly.

---

## Naming Conventions

### Use verb_noun Format
```
✅ create_issue, search_code, get_pull_request, deploy_service
❌ issue, code_search_handler, PRGetter, svc_deploy
```

### Be Specific, Not Generic
```
✅ search_github_issues    → Clear what it searches
❌ search                   → Search what?
❌ find_stuff               → Too vague
```

### Use Consistent Prefixes by Action Type

| Prefix | Usage | Example |
|--------|-------|---------|
| `get_` | Retrieve single item | `get_issue`, `get_user` |
| `list_` | Retrieve collection | `list_repos`, `list_branches` |
| `search_` | Query with filters | `search_issues`, `search_code` |
| `create_` | Create new item | `create_issue`, `create_branch` |
| `update_` | Modify existing item | `update_issue`, `update_settings` |
| `delete_` | Remove item | `delete_branch`, `delete_comment` |
| `run_` | Execute process | `run_tests`, `run_migration` |

---

## Schema Design

### Keep Parameters Flat (Max 6 Top-Level)

```json
// ✅ Good: Flat, clear parameters
{
  "properties": {
    "owner": { "type": "string" },
    "repo": { "type": "string" },
    "title": { "type": "string" },
    "priority": { "type": "string", "enum": ["low", "medium", "high"] }
  }
}

// ❌ Bad: Deeply nested
{
  "properties": {
    "repository": {
      "type": "object",
      "properties": {
        "owner": { "type": "string" },
        "name": { "type": "string" }
      }
    }
  }
}
```

### Use Enums Over Free-Form Strings

```json
// ✅ Constrained — AI gets it right every time
"state": { "type": "string", "enum": ["open", "closed", "all"] }

// ❌ Free-form — AI might say "opened", "Open", "active", etc.
"state": { "type": "string" }
```

### Provide Clear Descriptions for Every Parameter

```json
{
  "owner": {
    "type": "string",
    "description": "GitHub organization or username (e.g., 'microsoft', 'octocat')"
  },
  "since": {
    "type": "string",
    "description": "ISO 8601 date. Only return items after this date (e.g., '2026-01-01')"
  }
}
```

### Mark Required vs Optional Clearly

```json
{
  "required": ["owner", "repo", "title"],
  "properties": {
    "owner": { "type": "string" },
    "repo": { "type": "string" },
    "title": { "type": "string" },
    "body": { "type": "string", "description": "Optional detailed description" },
    "labels": { "type": "array", "description": "Optional labels to apply" }
  }
}
```

---

## Description Writing Guide

### Structure
```
[What the tool does]. [When to use it]. [Important constraints or defaults].
```

### Examples

```
✅ "Create a new issue in a GitHub repository. Use when the user wants to 
    report a bug, request a feature, or track a task. Requires owner and repo."

✅ "Run a read-only SQL query against the application database. Only SELECT 
    queries are allowed. Results are limited to 100 rows by default."

❌ "Creates an issue."  (Too terse — when should AI use this?)
❌ "This tool can be used to create issues in GitHub repositories 
    when the user expresses a desire to track work items."  (Too wordy)
```

---

## Key Takeaways

1. **verb_noun** naming: `create_issue`, not `issue` or `issueCreator`
2. **≤6 flat parameters**: No deep nesting
3. **Enums over strings**: Constrain values wherever possible
4. **Describe everything**: Every parameter needs a clear description
5. **Descriptions sell the tool**: The AI reads them to decide which tool to use

---

## Next Steps

- 🔗 [Error Handling](error-handling.md) — When things go wrong
- 🔗 [Common Mistakes](../07-anti-patterns/common-mistakes.md) — What to avoid
