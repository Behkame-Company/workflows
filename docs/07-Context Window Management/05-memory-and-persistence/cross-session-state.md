# Cross-Session State

> Bridging the gap between context windows for long-running projects.

---

## The Problem

Every new AI session starts with zero memory:

```
Session 1: Set up project, made great progress
[Session ends]
Session 2: "What project? What progress?"
```

Without cross-session state, each new session must:
- Re-discover the codebase
- Re-read configuration files
- Guess at what was done before
- Risk duplicating or contradicting previous work

---

## State Artifacts

The bridge between sessions is built from **state artifacts** — files that capture what the AI needs to know:

```
┌─────────────────────────────────────────────┐
│              STATE ARTIFACTS                 │
│                                             │
│  progress.txt     → What's done, what's next│
│  feature_list.json → Scope and status       │
│  git log          → What changed and when   │
│  init.sh          → How to start the app    │
│  CLAUDE.md        → Project conventions     │
│  activeContext.md  → Current working state   │
└──────────────────┬──────────────────────────┘
                   │
    ┌──────────────┼──────────────┐
    ▼              ▼              ▼
Session N-1    Session N     Session N+1
(writes)       (reads/writes) (reads)
```

---

## State Recovery Protocol

At the start of every session, the agent should:

```
1. pwd                          → Know where you are
2. Read progress.txt            → What was done, what's next
3. Read feature_list.json       → Overall scope and status
4. git log --oneline -20        → Recent changes
5. Read init.sh                 → How to start the app
6. Start dev server             → Verify it works
7. Run basic integration test   → Catch any regressions
8. Choose next feature          → Begin work
```

This protocol takes 30 seconds of model time but saves minutes of confused exploration.

---

## State Formats: JSON vs. Markdown

### JSON for Structured Data (Preferred for Status Tracking)
```json
{
  "category": "functional",
  "description": "User can log in with email and password",
  "steps": [
    "Navigate to login page",
    "Enter valid credentials",
    "Verify redirect to dashboard"
  ],
  "passes": false
}
```
**Why JSON**: Models are less likely to inappropriately modify or overwrite JSON than Markdown. Schema is enforced.

### Markdown for Prose/Notes
```markdown
## Session Notes — 2025-03-27
Working on payment integration. Stripe webhook handler
is complete but needs error handling for failed charges.
Next: implement refund flow.
```
**Why Markdown**: More natural for freeform context and reasoning.

---

## Git as State Tracker

Git provides automatic state tracking:
- **git log** — What was done and when
- **git diff** — What's changed since last session
- **git stash** — Save incomplete work
- **git revert** — Undo bad changes

```markdown
## Prompt for Git-Based State Recovery
At the start of each session:
1. Run `git log --oneline -20` to see recent work
2. Run `git status` to check for uncommitted changes
3. If there are uncommitted changes, review and commit or stash them
4. Read progress notes before starting new work
```

---

## Key Takeaways

1. **State artifacts are the bridge** — progress.txt, feature_list.json, git log
2. **Recovery protocol is essential** — Read state before doing anything
3. **JSON for status, Markdown for prose** — Use the right format
4. **Git is free state tracking** — Commit history is always available
5. **Write state before ending** — Future you will thank current you

---

## Next Steps

- 🔗 [Multi-Context Window Design](../06-multi-window-workflows/multi-context-window-design.md) — Full architecture
- 🔗 [Initializer Agent Pattern](../06-multi-window-workflows/initializer-agent-pattern.md) — First session setup
