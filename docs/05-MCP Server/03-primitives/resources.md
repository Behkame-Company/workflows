# Resources — Read-Only Data for AI Context

> Resources are the "nouns" of MCP — structured data the AI can read but not modify.

---

## What Are Resources?

Resources are **read-only data** exposed by MCP servers. They provide context that the AI needs to make informed decisions: file contents, database schemas, configuration, API documentation, user profiles.

```
Tools = Actions (verbs)     → "Create an issue"
Resources = Data (nouns)    → "Here's the database schema"
```

---

## Resource Definition

Each resource has a URI, name, description, and MIME type:

```json
{
  "uri": "postgres://mydb/schema",
  "name": "Database Schema",
  "description": "Current schema for the production database",
  "mimeType": "application/json"
}
```

---

## Resource Operations

### Discovery
```json
// Client → Server
{ "method": "resources/list", "id": 1 }

// Server → Client
{
  "result": {
    "resources": [
      {
        "uri": "config://app/settings",
        "name": "Application Settings",
        "mimeType": "application/json"
      },
      {
        "uri": "file:///src/schema.prisma",
        "name": "Prisma Schema",
        "mimeType": "text/plain"
      }
    ]
  }
}
```

### Reading
```json
// Client → Server
{
  "method": "resources/read",
  "id": 2,
  "params": { "uri": "config://app/settings" }
}

// Server → Client
{
  "result": {
    "contents": [{
      "uri": "config://app/settings",
      "mimeType": "application/json",
      "text": "{ \"database\": \"postgres\", \"port\": 5432, \"pool_size\": 10 }"
    }]
  }
}
```

### Subscriptions (Real-Time Updates)
```json
// Client → Server: subscribe to changes
{
  "method": "resources/subscribe",
  "params": { "uri": "logs://app/errors" }
}

// Server → Client: resource changed notification
{
  "method": "notifications/resources/updated",
  "params": { "uri": "logs://app/errors" }
}
```

---

## Resource vs Tool — When to Use Which

| Scenario | Use Resource | Use Tool |
|----------|-------------|---------|
| Show database schema | ✅ | ❌ |
| Run a SQL query | ❌ | ✅ |
| Display config file | ✅ | ❌ |
| Update config setting | ❌ | ✅ |
| List directory contents | ✅ | ❌ |
| Write a file | ❌ | ✅ |
| Show API documentation | ✅ | ❌ |
| Make an API call | ❌ | ✅ |

**Rule of thumb**: If it's read-only context, make it a resource. If it changes state, make it a tool.

---

## Resource URI Patterns

| Pattern | Example | Use Case |
|---------|---------|----------|
| `file://` | `file:///src/schema.prisma` | Local files |
| `postgres://` | `postgres://mydb/tables` | Database objects |
| `github://` | `github://repos/myorg/app/readme` | Repository data |
| `config://` | `config://app/settings` | Configuration |
| `logs://` | `logs://app/recent-errors` | Log data |
| `custom://` | `sentry://issues/recent` | Domain-specific |

---

## Resource Design Best Practices

1. **Right-size the data** — Don't dump entire databases. Summarize, paginate, or filter.
2. **Use descriptive URIs** — The AI uses URIs to understand what a resource contains.
3. **Set correct MIME types** — Helps the AI parse and understand the content format.
4. **Support subscriptions** — For data that changes frequently, let clients subscribe to updates.
5. **Keep resources focused** — One resource per logical data unit, not one giant dump.

---

## Key Takeaways

1. Resources provide **read-only context** to AI agents
2. They're identified by **URIs** and have **MIME types**
3. Use resources for data the AI needs to **read**, tools for actions it needs to **perform**
4. Resources can be **subscribed to** for real-time updates
5. Right-size your data — don't overwhelm the AI's context window
