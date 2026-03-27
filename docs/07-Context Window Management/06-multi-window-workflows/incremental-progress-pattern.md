# Incremental Progress Pattern

> How coding agents make steady, verifiable progress one feature at a time.

---

## The Pattern

Every coding agent window follows the same cycle:

```
GET BEARINGS → VERIFY → IMPLEMENT → TEST → COMMIT → UPDATE → REPEAT
```

---

## The Cycle in Detail

### Step 1: Get Bearings
```
Agent: I'll start by getting my bearings.
[Tool] pwd
[Tool] Read progress.txt
[Tool] Read feature_list.json
[Tool] git log --oneline -20
```

### Step 2: Verify Existing Work
```
Agent: Let me verify the app still works.
[Tool] Run init.sh (start dev server)
[Tool] Run basic integration test
[Tool] Check that previously passing features still pass
```

If broken → fix before implementing anything new.

### Step 3: Choose ONE Feature
```
Agent: Based on progress notes and the feature list,
       I'll work on "User can reset password" next.
```

**Critical**: One feature at a time. Not two. Not "and also..."

### Step 4: Implement
```
Agent: Implementing the password reset feature.
[Tool] Create src/auth/password-reset.ts
[Tool] Create src/routes/reset-password.ts
[Tool] Update src/auth/index.ts
```

### Step 5: Test Thoroughly
```
Agent: Let me verify this works end-to-end.
[Tool] Run unit tests
[Tool] Run integration test (simulate password reset flow)
[Tool] Verify via browser automation if applicable
```

Only mark the feature as passing after end-to-end verification.

### Step 6: Commit
```
git add .
git commit -m "feat: implement password reset flow

- Added password reset token generation
- Created reset password API endpoint
- Added email notification trigger
- All tests passing"
```

### Step 7: Update Progress
```
# Update progress.txt
=== Session 4 ===
- Implemented password reset feature
- All tests passing (28/45 features complete)
- Next: implement profile management

# Update feature_list.json
"description": "User can reset password"
"passes": true  ← Only after verification
```

### Step 8: Repeat or End Session
If context is running low → write detailed handoff notes and end.
If context remains → pick next feature and repeat.

---

## Common Failure Modes

| Failure | Cause | Prevention |
|---------|-------|-----------|
| Trying to do too much | Agent attempts multiple features | "Work on ONE feature at a time" |
| Marking features done prematurely | No end-to-end testing | "Only mark passing after full verification" |
| Half-implemented at session end | Ran out of context mid-feature | Monitor context budget; finish or revert |
| Breaking previous features | No regression testing | Always verify existing work first |

---

## Prompt for Coding Agent

```markdown
You are continuing work on a project. Follow this protocol:

1. Read progress.txt and git log to understand current state
2. Run init.sh to start the development server
3. Verify existing functionality works
4. Choose the highest-priority failing feature from feature_list.json
5. Implement ONE feature completely
6. Test it end-to-end (not just unit tests)
7. Only mark it as passing after verification succeeds
8. Commit with a descriptive message
9. Update progress.txt
10. If context allows, repeat from step 4

CRITICAL:
- Work on ONE feature at a time
- NEVER remove or edit feature descriptions in feature_list.json
- Only change the "passes" field from false to true
- Always commit before ending
- Write detailed progress notes
```

---

## Key Takeaways

1. **One feature at a time** — The single most important rule
2. **Verify before building** — Catch regressions immediately
3. **Test end-to-end** — Not just unit tests; full user flow
4. **Commit after each feature** — Clean state for next window
5. **Progress notes bridge sessions** — Write before ending

---

## Next Steps

- 🔗 [State Handoff Strategies](state-handoff-strategies.md) — Clean transitions
- 🔗 [Just-in-Time Retrieval](../07-context-retrieval/just-in-time-retrieval.md) — Efficient context loading
