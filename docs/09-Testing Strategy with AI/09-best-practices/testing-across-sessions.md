# Testing Across Sessions

> Maintaining test context, progress, and state across AI sessions — because context resets shouldn't mean losing track of your testing work.

---

## The Session Reset Problem

Every AI session starts with a blank slate. Without explicit persistence, you lose:

- Which tests exist and where they live
- Which tests pass, fail, or are still pending
- What testing strategy was agreed upon
- What edge cases were already identified
- What bugs were found and fixed via tests

```
Session 1: "Great, we have 12 tests, 10 passing, 2 need the email mock"
  --- close terminal / context window fills up ---
Session 2: "I don't see any test context. What would you like me to test?"
```

---

## Solution 1: tests.json Tracking File

The most direct solution — a JSON file that persists test state:

```json
{
  "meta": {
    "lastUpdated": "2024-03-15T14:22:00Z",
    "lastSession": "Added payment validation tests"
  },
  "tests": [
    { "name": "User registration with valid data", "file": "tests/auth.test.js", "status": "pass" },
    { "name": "Login with correct password", "file": "tests/auth.test.js", "status": "pass" },
    { "name": "Password reset email sends", "file": "tests/auth.test.js", "status": "pending", "notes": "Needs Mailgun mock" },
    { "name": "Order creation happy path", "file": "tests/orders.test.js", "status": "fail", "error": "Missing inventory check" }
  ]
}
```

See [Structured Test Tracking](structured-test-tracking.md) for the complete tests.json guide.

---

## Solution 2: Memory Files

Create a dedicated file that AI reads at session start:

### .ai/testing-memory.md

```markdown
# Testing Memory

## Current State
- Test framework: Jest + React Testing Library
- Coverage: 72% (target: 85%)
- Flaky tests: 2 (see quarantine section)

## What's Been Done
- [x] Auth module: full unit + integration tests
- [x] User CRUD: unit tests complete
- [ ] Payment module: needs integration tests
- [ ] Email service: needs mock setup
- [ ] Admin dashboard: no tests yet

## Known Issues
- `tests/payment.test.js:45` flaky on CI (race condition with Stripe mock)
- Coverage gap in error handling for `src/api/middleware.js`

## Testing Conventions
- Test files: `*.test.js` next to source files
- Fixtures: `tests/fixtures/`
- Factories: `tests/factories/`
- Use `createTestUser()` factory, not raw objects
- Mock external APIs, don't mock internal modules

## Next Session Should
1. Fix the Stripe mock race condition
2. Add integration tests for payment module
3. Increase coverage in middleware error handling
```

---

## Solution 3: Copilot Custom Instructions

Persist testing rules in your project's instruction file:

### .github/copilot-instructions.md

```markdown
## Testing Protocol

### Before Starting Any Coding Session
1. Read `tests.json` for current test status
2. Read `.ai/testing-memory.md` for testing context
3. Run `npm test` to verify baseline

### Test Conventions
- Framework: Jest + React Testing Library
- Test location: co-located with source (`*.test.js`)
- Always write tests before implementation (TDD)
- Run full suite after every change
- Update tests.json after test runs

### Current Testing Priorities
1. Payment module integration tests
2. Fix flaky Stripe mock test
3. Admin dashboard E2E tests

### Test Quality Rules
- Every test must have a descriptive name
- No `test.skip` without a linked issue
- No snapshot tests larger than 50 lines
- Minimum assertion per test: 1 meaningful assertion
```

---

## Solution 4: CLAUDE.md for Claude Code

```markdown
# CLAUDE.md

## Testing
- Run tests: `npm test`
- Run specific: `npm test -- --testPathPattern="auth"`
- Coverage: `npm test -- --coverage`

## Test State
- Read `tests.json` for current test status before any testing task
- Update `tests.json` after running tests
- Current focus: payment module needs integration tests

## Test Rules
- Write tests FIRST, then implement
- Never modify existing tests to make new code pass
- Always run full suite after changes
- If tests fail, fix implementation, not tests
```

---

## Solution 5: Git as Test State Persistence

Git history is always available and provides test state implicitly:

```bash
# What test files exist?
git ls-files "tests/**/*.test.*"

# What tests were added recently?
git log --oneline --diff-filter=A -- "tests/"

# What's the last test-related change?
git log --oneline -5 -- "tests/"

# Which test files have the most churn (likely problematic)?
git log --name-only --pretty=format: -- "tests/" | sort | uniq -c | sort -rn | head -10
```

### Git-Based Session Handoff

```bash
# End of session: commit test state
git add tests.json
git commit -m "test: update test tracking - 38/45 passing

Session summary:
- Added 5 payment validation tests
- Fixed auth token expiry test
- Pending: email mock setup for password reset tests"

# Start of next session: AI reads the commit message
git log --oneline -5 -- tests.json
```

---

## Solution 6: Memory Bank Pattern

A structured directory of context files:

```
.ai/
├── testing-memory.md       # Current test state and priorities
├── testing-decisions.md    # Why we chose certain approaches
├── testing-patterns.md     # Project-specific test patterns
└── session-log.md          # Brief log of what each session did
```

### session-log.md

```markdown
# Session Log

## 2024-03-15 14:00
- Added auth module unit tests (5 tests, all passing)
- Discovered edge case: Unicode usernames break validation
- Created issue #123 for Unicode fix

## 2024-03-15 16:30
- Added order creation integration tests (3 tests, 2 passing)
- Payment service mock needs work — test pending
- Updated tests.json

## 2024-03-16 09:00
- [Next session starts here]
```

---

## Session Handoff Checklist

Use this checklist when ending and starting AI sessions:

### Ending a Session

```markdown
## Session Exit Checklist
- [ ] All tests run (`npm test` output clean)
- [ ] tests.json updated with current status
- [ ] testing-memory.md updated with session progress
- [ ] Failing tests documented with error messages
- [ ] Pending work described clearly
- [ ] Git commit with descriptive test message
```

### Starting a Session

```markdown
## Session Start Checklist
- [ ] Read tests.json for test status
- [ ] Read testing-memory.md for context
- [ ] Check git log for recent test changes
- [ ] Run `npm test` to verify baseline
- [ ] Review pending items from last session
```

### AI Prompt for Session Start

```markdown
I'm continuing testing work on this project. Before we start:

1. Read `tests.json` for current test status
2. Read `.ai/testing-memory.md` for testing context
3. Run `npm test` and report results
4. Tell me:
   - How many tests pass/fail/pending?
   - What was the last session working on?
   - What should we focus on next?
```

---

## Comparing Approaches

| Approach | Persistence | Machine-Readable | Setup Effort | Best For |
|----------|------------|-----------------|-------------|----------|
| **tests.json** | ✅ File | ✅ JSON | Low | Test status tracking |
| **Memory files** | ✅ File | ⚠️ Markdown | Medium | Rich context |
| **Custom instructions** | ✅ File | ⚠️ Natural language | Low | Rules and conventions |
| **CLAUDE.md** | ✅ File | ⚠️ Natural language | Low | Claude Code projects |
| **Git history** | ✅ Git | ⚠️ Needs parsing | None | Always available |
| **Memory bank** | ✅ Directory | ⚠️ Mixed | High | Large projects |

**Recommendation:** Use **tests.json** (machine-readable status) + **one memory file** (human-readable context) + **custom instructions** (rules and conventions).

---

## Next Steps

- [Structured Test Tracking](structured-test-tracking.md) — deep dive on the tests.json format
- [Test-First Development with AI](test-first-development.md) — the workflow that generates tests to track
- [Coding Agent Test Verification](../06-ai-qa-workflows/agent-verification.md) — how agents use persistent test state
- [Testing Templates](../11-practical/templates.md) — templates for memory files and tracking

---

*Part of the [Testing Strategy with AI](../README.md) series.*
