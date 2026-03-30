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

Effective state management requires multiple layers — git history, feature lists, progress notes, active context, and decision logs. See [Cross-Session State](../05-memory-and-persistence/cross-session-state.md) for the full state management protocol.

---

## Key Takeaways

1. **State artifacts are cheap insurance** — 5 minutes of note-writing saves hours
2. **Feature list prevents scope loss** — Models can't forget what's not done
3. **Clean commits are checkpoints** — Always recoverable
4. **Handoff protocol is mandatory** — Never end a session without writing state
5. **All 5 layers work together** — Git + features + progress + context + decisions
