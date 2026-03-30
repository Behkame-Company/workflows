# Memory Tool Guide

> Anthropic's memory tool — persistent storage that extends beyond the context window.

---

## What Is the Memory Tool?

The memory tool is a client-side tool that lets Claude store and retrieve information in a `/memories` directory. It persists between sessions, enabling:

- **Cross-session learning** — Remember decisions from yesterday
- **Knowledge base building** — Accumulate project knowledge
- **State persistence** — Pick up where you left off
- **Context offloading** — Store info outside the context window

---

## How It Works

```
Turn 1: Claude checks /memories directory
Turn 2: Claude reads relevant memory files
Turn 3: Claude uses memories to inform current work
Turn 4: Claude updates memories with new information
```

### Operations

| Command | Description |
|---------|-------------|
| `view` | List files or read file contents |
| `create` | Create a new memory file |
| `str_replace` | Update content in a file |
| `insert` | Insert text at a specific line |
| `delete` | Delete a file or directory |
| `rename` | Move/rename a file |

---

## Example Memory File Structure

```
/memories/
├── project-context.md      # Tech stack, architecture
├── decisions.md             # Key decisions with rationale
├── patterns.md              # Code patterns to follow
├── current-task.md          # What we're working on
├── known-issues.md          # Bugs and workarounds
└── review-feedback.md       # Recurring review feedback
```

### project-context.md
```markdown
# Project Context
- Framework: Next.js 14 (App Router)
- Language: TypeScript strict
- DB: PostgreSQL + Prisma
- Auth: JWT with RS256
- Deploy: AWS ECS
```

### decisions.md
```markdown
# Decisions Log

## 2025-03-15: Auth approach
Decision: JWT with RS256 + refresh tokens
Rationale: Stateless auth for microservices
Trade-off: More complex than sessions

## 2025-03-20: Database
Decision: PostgreSQL over MongoDB
Rationale: Relational data model, ACID compliance
```

---

## Best Practices

### 1. Keep Memories Organized
- One file per topic (not one giant file)
- Use descriptive filenames
- Delete outdated memories

### 2. Update, Don't Duplicate
- Update existing files rather than creating new ones
- Consolidate related information periodically

### 3. Memory Should Be High-Signal
- Store decisions, not discussions
- Store patterns, not individual code snippets
- Store state, not full conversation history

### 4. Prompt for Memory Usage
```markdown
You have a memory tool. At the start of each session:
1. Read your memory files for project context
2. Update memories with new information as you work
3. Before ending a session, update current-task.md

Keep memories organized and up-to-date.
```

---

## Key Takeaways

1. **Persists between sessions** — The model remembers across conversations
2. **Client-side** — You control storage (file, database, cloud)
3. **Organized > comprehensive** — Keep files focused and current
4. **Update continuously** — Not just at start/end of session
5. **Pair with compaction** — Memory survives context resets
