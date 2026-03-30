# The Stale Context Trap

> When outdated information in the context window actively misleads the model.

---

## What Is Stale Context?

Stale context is information that was once accurate but is no longer true. It stays in the context window and misleads the model.

---

## Common Sources of Stale Context

### 1. Outdated Tool Results
```
Turn 5:  AI reads config.json → { "port": 3000 }
Turn 15: Developer changes port to 8080 (outside AI session)
Turn 20: AI references config.json from Turn 5 → uses port 3000 ❌
```

### 2. Superseded Decisions
```
Turn 3:  "Let's use MongoDB for the database"
Turn 12: "Actually, let's switch to PostgreSQL"
Turn 25: Model references Turn 3's decision → uses MongoDB patterns ❌
```

### 3. Reverted Code
```
Turn 8:  AI writes function A (approach 1)
Turn 10: Developer reverts to approach 2
Turn 15: AI assumes approach 1 is still in place ❌
```

### 4. Completed TODOs
```
Turn 5:  "TODO: Add input validation"
Turn 12: Validation was added
Turn 20: Model adds validation again (duplication) ❌
```

---

## Why Stale Context Is Dangerous

| Issue | Impact |
|-------|--------|
| **Silent failures** | Model generates code against wrong assumptions |
| **Inconsistency** | New code contradicts recent changes |
| **Wasted effort** | Model re-implements completed work |
| **Debugging chaos** | Mixing old and new context creates confusion |

---

## Prevention Strategies

### 1. Clear Old Tool Results
After using tool output, clear it:
```
Turn 5: [read_file result: 1,500 tokens] → Process it
Turn 6: [Replace with: "Read config.json — port 3000, env dev"]
```
If the model needs to re-read, it gets the current version.

### 2. Update State Files
Keep progress.txt and activeContext.md current:
```
After changing a decision:
  Update decisions.md → Remove old decision, add new one
  
After completing a TODO:
  Update progress.md → Mark as done
```

### 3. Re-Read Before Referencing
```markdown
# In system prompt:
Before referencing a file you read earlier in this session,
re-read it to get the current version. Files may have
changed since your last read.
```

### 4. Compact Completed Work
Once a sub-task is done, compact those turns:
```
Turns 5-15: "Implemented JWT auth, all tests passing"
(Instead of keeping all 10 turns of exploration and implementation)
```

---

## Key Takeaways

1. **Stale context causes silent errors** — The model trusts old information
2. **Tool results go stale fastest** — Files change outside the session
3. **Clear or update, don't keep** — Stale data is worse than no data
4. **Re-read before referencing** — Prompt the model to verify current state
5. **State files must stay current** — Update progress/decisions actively
