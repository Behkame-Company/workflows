# Poor State Management

> When agents lose track of what they've done, what they're doing, and what comes next.

---

## What Goes Wrong

Poor state management manifests as agents that:
- **Re-do completed work** — Implement a feature that's already done
- **Declare victory prematurely** — "The project is complete" (it's 20% done)
- **Leave broken state** — Half-implemented feature, uncommitted changes
- **Lose architectural coherence** — Each session makes contradictory decisions
- **Waste context re-discovering** — Spend 30% of each window figuring out what happened

---

## Root Causes

### 1. No Progress Tracking
```
❌ No progress file
   Agent: "Let me look around and figure out what to do..."
   [Wastes 10K tokens exploring]

✅ With progress.txt
   Agent: "I see we completed auth and API. Next: payment module."
   [Starts productive work immediately]
```

### 2. No Feature Scope
```
❌ No feature list
   Agent: "Everything seems to work, I think we're done."
   Reality: 30 features remain

✅ With feature_list.json
   Agent: "15/45 features passing. Next: password reset."
```

### 3. No Clean Commits
```
❌ No commits between features
   Agent in Window 3: "There are uncommitted changes...
   I'm not sure what state this is in..."

✅ Clean commits after each feature
   Agent in Window 3: git log → sees exactly what was done
```

### 4. No Handoff Protocol
```
❌ Session ends abruptly
   Next session: "What was happening here?"

✅ Session end protocol writes handoff notes
   Next session: Reads notes → continues immediately
```

---

## The State Management Stack

```
Layer 1: Git History         → What changed and when
Layer 2: Feature List        → What's done and what's not
Layer 3: Progress Notes      → What happened each session
Layer 4: Active Context      → Current working state
Layer 5: Decision Log        → Why things were done this way
```

All five layers together give a new agent session complete state recovery.

---

## Prevention: The State Protocol

```markdown
## Agent State Protocol

### Start of Session
1. Read progress.txt → Know what was done
2. Read feature_list.json → Know the scope
3. git log --oneline -20 → See recent changes
4. Start dev server → Verify it works
5. Run basic test → Catch regressions

### During Session
6. Work on ONE feature at a time
7. Commit after each feature
8. Update feature_list.json status
9. Update progress.txt with notes

### End of Session
10. Finish or revert current work (never leave broken)
11. Write handoff notes to progress.txt
12. Final commit: "session-end: update progress"
13. Verify app is in working state
```

---

## Key Takeaways

1. **State artifacts are cheap insurance** — 5 minutes of note-writing saves hours
2. **Feature list prevents scope loss** — Models can't forget what's not done
3. **Clean commits are checkpoints** — Always recoverable
4. **Handoff protocol is mandatory** — Never end a session without writing state
5. **All 5 layers work together** — Git + features + progress + context + decisions

---

## Next Steps

- 🔗 [Templates](../11-practical/templates.md) — Ready-to-use state management files
- 🔗 [State Handoff Strategies](../06-multi-window-workflows/state-handoff-strategies.md) — Clean transitions
