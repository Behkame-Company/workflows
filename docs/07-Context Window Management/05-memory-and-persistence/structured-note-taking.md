# Structured Note-Taking

> Teaching AI agents to document their own work — the most powerful context management technique.

---

## The Concept

Structured note-taking is a technique where the agent regularly writes notes persisted outside the context window. These notes are retrieved into context when needed, enabling:

- **Persistent memory** without growing the context window
- **Progress tracking** across complex tasks
- **Dependency tracking** that survives compaction
- **Decision logging** for audit and continuity

---

## The Pattern

```
Agent works → Makes progress → Writes notes → Continues working
                                    ↓
                              [notes.md file]
                                    ↓
Next session → Reads notes → Knows where to resume
```

---

## Types of Notes

### 1. Progress Notes
```markdown
# Progress Log

## Session 2025-03-27
- ✅ Set up Express server with TypeScript
- ✅ Created user model with Prisma
- ✅ Implemented JWT authentication
- 🔄 In progress: Role-based authorization
- ⬜ TODO: Rate limiting middleware
- ⬜ TODO: API documentation
```

### 2. Decision Notes
```markdown
# Architectural Decisions

## ADR-001: Auth Strategy
- Decided: JWT with RS256
- Alternatives: Sessions, OAuth2 only
- Rationale: Stateless for microservices
- Date: 2025-03-15
```

### 3. Bug Tracker
```markdown
# Known Issues

## BUG-001: CORS on WebSocket upgrade
- Status: Open
- Symptom: WebSocket fails in production
- Root cause: Nginx not forwarding upgrade headers
- Workaround: Direct connection bypassing Nginx
```

### 4. Context Notes (for next session)
```markdown
# Handoff Notes

## Current State
Working on: RBAC middleware in src/auth/rbac.ts
Last action: Created role enum, started permission check
Next step: Wire middleware into Express routes
Blockers: None
Critical files: src/auth/rbac.ts, src/auth/types.ts, src/routes/api.ts
```

---

## Real-World Example: Claude Playing Pokémon

Anthropic's Claude playing Pokémon demonstrates the power of structured note-taking:
- Agent maintains precise tallies across thousands of game steps
- Tracks training progress: "Pikachu gained 8 levels toward target of 10"
- Develops maps of explored regions from notes
- Maintains combat strategy notes
- Continues multi-hour tasks after context resets

Without notes, this would be impossible — the context window can't hold thousands of game steps.

---

## Implementation

### Prompt the Agent
```markdown
## Note-Taking Protocol
As you work, maintain structured notes:

1. **progress.txt**: After completing each sub-task, update this file
2. **decisions.md**: Log any architectural or design decisions
3. **issues.md**: Track bugs discovered during work

At the end of a session, write a handoff note summarizing:
- What was completed
- What's in progress
- What's blocked
- What to do next
```

### File-Based vs. Tool-Based
| Approach | Pros | Cons |
|----------|------|------|
| **File system** (write to disk) | Simple, versioned with git | Requires file access |
| **Memory tool** | Structured API, cross-session | Requires tool setup |
| **Database** | Queryable, scalable | More complex |

---

## Key Takeaways

1. **Notes persist, context doesn't** — Write it down before it's compacted
2. **Types matter** — Progress, decisions, bugs, handoff
3. **Prompt for note-taking** — Models don't do this automatically
4. **Update continuously** — Not just at session end
5. **Notes enable multi-session work** — The bridge between context windows

---

## Next Steps

- 🔗 [Memory Bank Pattern](memory-bank-pattern.md) — Cline's approach
- 🔗 [Cross-Session State](cross-session-state.md) — Full state management
