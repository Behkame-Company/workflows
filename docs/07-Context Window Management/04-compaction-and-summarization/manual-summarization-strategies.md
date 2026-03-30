# Manual Summarization Strategies

> DIY context compression techniques for when you need more control.

---

## Strategy 1: Tool Result Clearing

The simplest form of compaction — replace old tool outputs with brief summaries:

```
Before:
  Turn 5: [read_file result — 1,500 tokens of code]
  Turn 6: AI: "I've analyzed the file. The auth module uses JWT..."

After clearing:
  Turn 5: [Tool result cleared: "Read src/auth/jwt.ts (200 lines)"]
  Turn 6: AI: "I've analyzed the file. The auth module uses JWT..."
```

**When to clear**: After the AI has processed and responded to a tool result, the raw result is rarely needed again.

---

## Strategy 2: Conversation Summarization

Periodically replace old conversation turns with a summary:

```
Before (Turns 1-20, ~40K tokens):
  Turn 1: User asked about project setup
  Turn 2: AI set up Node.js project
  Turn 3: User asked for auth module
  ...
  Turn 20: AI fixed test failures

After summarization (~2K tokens):
  [Summary]: "Set up Node.js project with Express, implemented
   JWT authentication with refresh tokens, created user CRUD
   API with Prisma ORM, fixed 3 test failures in auth module.
   Current state: All tests passing, auth module complete."
```

### Implementation Pattern
```python
def summarize_conversation(messages, keep_recent=5):
    old_messages = messages[:-keep_recent]
    recent_messages = messages[-keep_recent:]
    
    summary = client.messages.create(
        model="claude-haiku-4-5",  # Cheap model for summarization
        messages=[{
            "role": "user",
            "content": f"Summarize this conversation concisely, preserving:\n"
                       f"1. Key decisions made\n"
                       f"2. Current state of work\n"
                       f"3. Unresolved issues\n"
                       f"4. File changes made\n\n"
                       f"Conversation:\n{format_messages(old_messages)}"
        }]
    )
    
    return [{"role": "user", "content": f"[Previous context summary]: {summary}"}] + recent_messages
```

---

## Strategy 3: Progressive Summarization

Summarize in tiers — recent detail, older summary, oldest high-level:

```
┌──────────────────────────────────────────┐
│ Tier 1: Last 5 turns (full detail)       │  ~10K tokens
├──────────────────────────────────────────┤
│ Tier 2: Turns 6-20 (paragraph summary)   │  ~2K tokens
├──────────────────────────────────────────┤
│ Tier 3: Turns 21+ (bullet-point summary) │  ~500 tokens
└──────────────────────────────────────────┘
```

---

## Strategy 4: Selective Retention

Keep only messages matching certain criteria:
- Messages containing architectural decisions → Keep
- Messages with error resolution → Keep
- Messages about completed sub-tasks → Summarize
- Messages with exploration/dead ends → Remove

---

## Strategy 5: State Snapshot

Instead of summarizing the conversation, capture the current state:

```markdown
## Current State Snapshot
- **Task**: Building user authentication module
- **Completed**: JWT generation, token refresh, password hashing
- **In Progress**: Role-based authorization middleware
- **Blocked**: None
- **Files Modified**: src/auth/jwt.ts, src/auth/middleware.ts, tests/auth.test.ts
- **Key Decisions**: Using RS256 for JWT, bcrypt cost factor 12
- **Next Steps**: Implement RBAC middleware, add rate limiting
```

This snapshot replaces the entire conversation with a structured state.

---

## Key Takeaways

1. **Tool result clearing is the easiest win** — High impact, low effort
2. **Summarize old turns** — Use a cheap model for compression
3. **Progressive tiers** — Most detail for recent, least for oldest
4. **State snapshots** — Replace conversation with structured state
5. **Preserve decisions and blockers** — Don't lose what matters
