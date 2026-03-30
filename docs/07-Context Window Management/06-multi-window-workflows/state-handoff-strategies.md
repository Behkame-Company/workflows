# State Handoff Strategies

> How to leave a clean, recoverable state when transitioning between context windows.

---

## The Handoff Challenge

When a context window ends (compaction, session end, or fresh start), the next window needs to reconstruct the full project state from artifacts alone.

**Bad handoff**: Next agent spends 30% of its context re-discovering state.
**Good handoff**: Next agent spends 5% of its context recovering and starts work immediately.

---

## Handoff Artifact Checklist

Before ending a session, ensure these artifacts are up to date:

| Artifact | Format | Contains |
|----------|--------|----------|
| **progress.txt** | Markdown | Session summary, what's done, what's next |
| **feature_list.json** | JSON | Feature scope and completion status |
| **Git history** | Git | Descriptive commits for each change |
| **activeContext.md** | Markdown | Current working state details |
| **init.sh** | Shell script | How to start the application |

---

## Clean Handoff Protocol

### Before Ending (Write Phase)

```markdown
1. Finish or revert current work
   - If feature is complete → commit it
   - If feature is half-done → revert to last commit
   - NEVER leave the codebase in a broken state

2. Update progress.txt:
   - What was completed this session
   - What's in progress (if anything)
   - What's blocked
   - Recommended next steps

3. Commit everything:
   git add .
   git commit -m "session-end: update progress notes"

4. Verify the app works:
   - Run tests
   - Start the server
   - Basic smoke test
```

### After Starting (Read Phase)

```markdown
1. Orient yourself:
   - pwd
   - Read progress.txt
   - git log --oneline -20

2. Understand scope:
   - Read feature_list.json
   - Count completed vs remaining features

3. Verify state:
   - Run init.sh
   - Run basic integration test
   - If broken → fix before continuing

4. Choose next work:
   - Pick highest-priority failing feature
   - Begin implementation
```

---

## Handoff Quality Levels

### Level 1: Minimal (Poor)
```
progress.txt: "Worked on auth stuff."
```
Next agent: Confused, wastes time exploring.

### Level 2: Adequate
```
progress.txt:
- Completed: JWT authentication
- Next: Implement password reset
```
Next agent: Knows the general direction.

### Level 3: Excellent
```
progress.txt:
=== Session 3 (2025-03-27) ===

Completed:
- JWT authentication with RS256 (src/auth/jwt.ts)
- Token refresh endpoint (src/routes/auth/refresh.ts)
- Auth middleware (src/auth/middleware.ts)
- Tests: 12 passing (tests/auth.test.ts)

State:
- All tests passing
- Server runs clean on port 3000
- Feature list: 15/45 features complete

Next priority:
- Implement password reset (feature_list.json #16)
- Files to modify: src/routes/auth/, src/services/email.ts
- Dependencies: Need to configure SMTP (use Mailtrap for dev)

Known issues:
- Token expiry is set to 1h, may need adjustment later
```
Next agent: Picks up instantly, starts building immediately.

---

## Key Takeaways

1. **Never leave broken state** — Commit or revert, no in-between
2. **Progress notes are the lifeline** — Be specific, not vague
3. **Git is your safety net** — Descriptive commits enable recovery
4. **Verify before handing off** — Run tests to confirm working state
5. **Level 3 handoffs save the most time** — Invest 2 minutes to save 20
