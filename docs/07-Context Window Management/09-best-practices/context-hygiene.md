# Context Hygiene

> Maintaining clean, relevant context throughout a session — preventing drift and pollution.

---

## What Is Context Hygiene?

Context hygiene is the ongoing practice of keeping the context window clean, relevant, and well-organized. Like code hygiene (formatting, linting), context hygiene prevents gradual degradation.

---

## Hygiene Practices

### 1. Clear Stale Tool Results
```
Turn 5:  Read large file → 1,500 tokens
Turn 10: File contents no longer relevant
Action:  Clear the tool result (replace with summary)
```

### 2. Start New Conversations for New Tasks
```
❌ Same thread for 5 unrelated tasks → 50K tokens of noise
✅ New conversation per task → Each starts with clean context
```

### 3. Don't Repeat Context
```
❌ "As I mentioned earlier, the project uses React..."
   (Information is still in context from earlier)

✅ Move on — the model already knows this from earlier context
```

### 4. Close Irrelevant Files (IDE-based tools)
For tools like Copilot that use open files as context:
```
❌ 20 tabs open, only 2 relevant → 18 files of noise
✅ Close irrelevant tabs → Only relevant files in context
```

### 5. Prune Dead-End Conversations
If you tried an approach and it failed:
```
❌ Keep the entire failed exploration in context
✅ Start fresh with: "Approach X didn't work because Y.
   Let's try approach Z instead."
```

---

## Session Lifecycle

```
START OF SESSION
  ├── Load minimal context (instructions, project overview)
  ├── Read state artifacts (progress, memory)
  
DURING SESSION
  ├── Clear tool results after processing
  ├── Summarize completed sub-tasks
  ├── Remove dead-end exploration
  ├── Re-inject critical instructions if context is long
  
END OF SESSION
  ├── Write progress notes
  ├── Commit all work
  ├── Update state artifacts
```

---

## Re-Injection Pattern

For long sessions, re-state critical instructions periodically:

```markdown
## Reminder
Before continuing, remember:
- We're using TypeScript strict mode
- All API endpoints must validate input with Zod
- Error responses use the standard format
- We're NOT using ORM for this migration query
```

This combats the "lost in the middle" effect.

---

## Context Health Indicators

| Indicator | Healthy | Unhealthy |
|-----------|---------|-----------|
| Model follows instructions | Consistently | Drifting from rules |
| Model remembers recent decisions | Yes | Repeating questions |
| Model references correct files | Yes | Hallucinating file names |
| Response quality | Stable | Degrading over time |
| Model stays on task | Yes | Wandering to unrelated topics |

If you see unhealthy indicators → compact, re-inject instructions, or start a new session.

---

## Key Takeaways

1. **Clear stale tool results** — The #1 low-hanging fruit
2. **New conversation for new tasks** — Don't pollute with old context
3. **Re-inject instructions periodically** — Combat the lost-middle effect
4. **Watch for health indicators** — Catch drift early
5. **Session end = state save** — Write notes before closing

---

## Next Steps

- 🔗 [Debugging Context Issues](debugging-context-issues.md) — When things go wrong
- 🔗 [Tool-Specific Strategies](tool-specific-strategies.md) — Per-tool guidance
