# Multi-Context Window Design

> Architecture for tasks that span multiple context windows — hours or days of work.

---

## The Challenge

Most complex development tasks exceed a single context window:

```
Task: "Build a full-stack e-commerce app"
Estimated effort: 200+ tool calls, 500K+ tokens
Single context window: 200K-1M tokens
Reality: Needs 3-10 context windows
```

Without multi-window design, each new window starts from scratch.

---

## The Two-Agent Architecture

Anthropic's proven pattern uses two specialized agent prompts:

```
┌─────────────────────────────────────────┐
│  INITIALIZER AGENT (Window 1 only)       │
│                                          │
│  Sets up:                                │
│  • Feature list (scope of work)          │
│  • Test framework                        │
│  • Init script (how to run the app)      │
│  • Progress file (tracking)              │
│  • Initial git commit                    │
└──────────────────┬───────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────┐
│  CODING AGENT (Windows 2, 3, 4, ...)     │
│                                          │
│  Each window:                            │
│  1. Reads progress file + git log        │
│  2. Runs init.sh + basic tests           │
│  3. Picks ONE feature to implement       │
│  4. Implements and tests it              │
│  5. Commits with descriptive message     │
│  6. Updates progress file                │
│  7. Repeats or ends session              │
└─────────────────────────────────────────┘
```

---

## Key Design Principles

### 1. Incremental Progress
Work on ONE feature at a time, not everything at once:
```
❌ "Build the entire auth system"
✅ "Implement user registration endpoint"
   → Test → Commit → Next feature
```

### 2. Clean State Between Windows
Each window ends with:
- All work committed to git
- Progress notes updated
- No half-implemented features
- App in a working state

### 3. Verification Before Building
Each new window starts by verifying the app works:
```
1. Start dev server
2. Run basic integration test
3. If broken → fix before continuing
4. If working → pick next feature
```

### 4. Feature List as Contract
```json
[
  { "description": "User registration", "passes": true },
  { "description": "User login", "passes": true },
  { "description": "Password reset", "passes": false },
  { "description": "Profile management", "passes": false }
]
```
This prevents the model from declaring victory early or losing scope.

---

## Choosing: Fresh Window vs. Compaction

| Approach | When to Use |
|----------|------------|
| **Fresh window** | Complex project with many features; model discovers state from filesystem |
| **Compaction** | Continuous conversation; maintaining conversational flow matters |
| **Hybrid** | Compact within a window; start fresh across sessions |

For most development tasks, **fresh windows with state artifacts** outperform compaction because models excel at discovering state from files and git.

---

## Key Takeaways

1. **Two agents: initializer + coding** — Different prompts for different jobs
2. **One feature per window** — Don't try to do everything at once
3. **Clean commits between features** — Never leave half-done work
4. **Verify before building** — Each window checks existing work first
5. **Feature list prevents scope loss** — Models can't declare victory early

---

## Next Steps

- 🔗 [Initializer Agent Pattern](initializer-agent-pattern.md) — Setting up Window 1
- 🔗 [Incremental Progress Pattern](incremental-progress-pattern.md) — Subsequent windows
