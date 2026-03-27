# Compaction Tuning

> Balancing recall vs. compression — what to keep, what to drop, and how to validate.

---

## The Recall-Compression Spectrum

```
Full Recall                                     Maximum Compression
(keep everything)                               (keep nothing)
├──────────┼──────────┼──────────┼──────────────┤
  Tool result   Progressive    State      Single
  clearing      summarization  snapshot   sentence
  (safest)      (balanced)     (compact)  (lossy)
```

---

## What to Always Keep

| Category | Why | Example |
|----------|-----|---------|
| **Architectural decisions** | Affect all future code | "Using microservices with API gateway" |
| **Active constraints** | Prevent regressions | "Must support IE11" |
| **Current task state** | Continuity | "Implementing feature B, step 3 of 5" |
| **Unresolved errors** | Need to be fixed | "CORS issue on /api/auth still open" |
| **Key file paths** | Navigation | "Auth logic in src/auth/middleware.ts" |

## What to Safely Drop

| Category | Why Safe | Example |
|----------|---------|---------|
| **Raw tool outputs** | Already processed | Full file contents after analysis |
| **Exploration dead ends** | Not relevant anymore | "Tried approach X, didn't work" |
| **Completed sub-tasks** | No longer active | "Set up linting ✅" → just the ✅ |
| **Verbose explanations** | Key point captured | Long reasoning behind a decision |
| **Repeated context** | Redundant | Same info stated multiple ways |

---

## Validation: Did Compaction Lose Something?

After compaction, test the model:

### Quick Recall Test
```
Ask: "What authentication approach are we using?"
Expected: "JWT with RS256, refresh tokens, bcrypt password hashing"

Ask: "What files have we modified?"
Expected: Lists recently modified files

Ask: "Are there any unresolved issues?"
Expected: Mentions known blockers
```

### Red Flags After Compaction
- Model asks questions it already answered
- Model contradicts earlier decisions
- Model loses track of which files to edit
- Model re-does work that's already complete

---

## Tuning Compaction Prompts

### Default (Anthropic)
Works well for general conversations. May over-compress technical details.

### Custom Prompt for Development Sessions
```
Summarize this conversation, preserving:
1. All architectural and design decisions with rationale
2. Current task state (what's done, what's in progress, what's next)
3. Unresolved errors or blockers
4. File paths that were modified or need modification
5. Key technical constraints or requirements
6. Test results and coverage information

Be concise but do not omit any technical details that would
be needed to continue this work session.
```

---

## Compaction Frequency

| Session Type | Compact Every | Rationale |
|-------------|---------------|-----------|
| Quick task (30 min) | Rarely/never | Fits in one window |
| Medium task (2 hrs) | 1-2 times | At natural breakpoints |
| Long session (4+ hrs) | 3-5 times | Multiple feature completions |
| Multi-day project | Per session | Clean start each day |

---

## Key Takeaways

1. **Always keep decisions and constraints** — They affect all future work
2. **Safely drop raw tool outputs** — After they've been processed
3. **Validate after compaction** — Ask recall questions to check
4. **Custom prompts for dev sessions** — Default may miss technical details
5. **Compact at natural breakpoints** — After completing a feature/task

---

## Next Steps

- 🔗 [Memory Tool Guide](../05-memory-and-persistence/memory-tool-guide.md) — Persist beyond compaction
- 🔗 [Cross-Session State](../05-memory-and-persistence/cross-session-state.md) — Bridging windows
