# Regression Testing Strategy

> Preventing AI-introduced regressions through disciplined test execution, smart test selection, and a culture that never lets AI hide bugs by modifying existing tests.

---

## Why AI Changes Increase Regression Risk

AI-generated code can introduce regressions in ways that surprise experienced developers:

- **Subtle behavioral changes** — AI may rewrite a function that looks correct but handles edge cases differently
- **Unintended side effects** — AI modifying one module can break dependent modules it wasn't asked about
- **Plausible-looking bugs** — AI code passes cursory review because it reads well, even when it's wrong
- **Test modification** — AI may "fix" failing tests instead of fixing code, masking real regressions

```
Traditional Development:        AI-Assisted Development:
─────────────────────────       ─────────────────────────
Change → Test → Ship            Change → Test → Ship
  │                               │
  └─ Developer knows what        └─ AI changes may have
     they changed                    unexpected scope
```

---

## The Golden Rule

> **Never let AI modify existing tests to make its code pass.**
>
> If existing tests fail after AI generates code, the code is wrong — not the tests.

This is the single most important regression prevention rule. AI will naturally try to "fix" test failures by adjusting assertions. This hides bugs rather than fixing them.

```markdown
# In your copilot-instructions.md or CLAUDE.md:
## Testing Rules
- NEVER modify existing tests to make new code pass
- If existing tests fail, fix the implementation, not the tests
- Only modify tests when the SPECIFICATION changes (not the implementation)
- Always run the full test suite after any code change
```

---

## Full Suite Execution After AI Changes

Always run the **complete** test suite after AI changes — not just the new tests:

```bash
# ❌ Wrong: Only running new tests
npm test -- --testPathPattern="new-feature"

# ✅ Right: Running the full suite
npm test

# ✅ Better: Full suite with coverage comparison
npm test -- --coverage --coverageReporters=json-summary
```

### Why "Just the New Tests" Is Dangerous

```javascript
// AI generates a utility function
export function formatCurrency(amount) {
  return `$${amount.toFixed(2)}`;  // Looks correct
}

// AI writes a test for it — passes ✅
test('formats currency', () => {
  expect(formatCurrency(10)).toBe('$10.00');
});

// BUT: An existing function USED to call the old formatCurrency
// which returned { symbol: '$', value: '10.00' }
// 14 existing tests now fail — you'd never know if you only ran the new test
```

---

## Regression Test Selection

Not all tests are equally likely to catch AI-related regressions. Prioritize:

### High-Priority Regression Tests

| Test Category | Why It Catches AI Regressions |
|--------------|-------------------------------|
| **Integration tests** | AI often breaks module boundaries |
| **Contract tests** | AI may change function signatures subtly |
| **Edge case tests** | AI frequently misses boundary conditions |
| **Error handling tests** | AI tends to implement the happy path only |
| **Backward compatibility** | AI may break existing API consumers |

### Risk-Based Test Selection

```python
# Example: Categorizing tests by regression risk
REGRESSION_RISK_MAP = {
    "high": [
        "tests/integration/",        # Cross-module boundaries
        "tests/api/contracts/",       # Public API contracts
        "tests/edge_cases/",          # Boundary conditions
    ],
    "medium": [
        "tests/unit/",               # Individual functions
        "tests/utils/",              # Shared utilities
    ],
    "low": [
        "tests/ui/snapshots/",       # Visual snapshots
        "tests/performance/",        # Performance benchmarks
    ],
}

# After AI changes, always run high + medium
# Run low on CI or before merge
```

---

## Building a Regression Safety Net

### Layer 1: Pre-Commit Checks

```yaml
# .husky/pre-commit or similar
#!/bin/sh
# Run tests related to changed files
npx jest --findRelatedTests $(git diff --cached --name-only)
```

### Layer 2: CI Pipeline Full Suite

```yaml
# .github/workflows/regression.yml
name: Regression Tests
on: [push, pull_request]

jobs:
  regression:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm test -- --ci --coverage
      - name: Compare coverage
        run: |
          # Fail if coverage dropped
          npx coverage-diff --baseline=main --threshold=0
```

### Layer 3: Smoke Tests After Deploy

```javascript
// tests/smoke/critical-paths.test.js
describe('Critical Path Smoke Tests', () => {
  test('user can log in', async () => {
    const response = await request(app).post('/login').send(credentials);
    expect(response.status).toBe(200);
  });

  test('user can create order', async () => {
    const response = await request(app).post('/orders').send(orderData);
    expect(response.status).toBe(201);
  });

  test('payment processing works', async () => {
    const response = await request(app).post('/payments').send(paymentData);
    expect(response.status).toBe(200);
  });
});
```

---

## Tracking Regression Frequency

Monitor regressions over time to identify patterns:

```markdown
## Regression Tracking Template

| Date       | PR     | Regression Description          | Root Cause        | AI-Involved? | Tests Added |
|-----------|--------|--------------------------------|-------------------|--------------|-------------|
| 2024-01-15 | #234   | Login broke after auth refactor | Changed return type | Yes         | 3           |
| 2024-01-22 | #251   | CSV export missing headers     | AI missed edge case | Yes         | 2           |
| 2024-02-01 | #267   | API rate limit bypassed        | AI removed guard   | Yes         | 1           |
```

### Metrics to Track

```python
# regression_metrics.py
regression_metrics = {
    "total_regressions_per_month": 0,
    "ai_caused_regressions": 0,        # vs human-caused
    "mean_time_to_detect": "hours",     # How fast caught
    "mean_time_to_fix": "hours",        # How fast resolved
    "regression_by_category": {
        "api_contract": 0,
        "edge_case": 0,
        "integration": 0,
        "performance": 0,
    },
    "tests_added_per_regression": 0,    # Growing safety net
}
```

---

## Regression Prevention Checklist

Before merging any AI-generated code:

```markdown
- [ ] Full test suite passes (not just new tests)
- [ ] No existing tests were modified by AI
- [ ] Coverage did not decrease
- [ ] Integration tests pass
- [ ] Contract/API tests pass
- [ ] Edge case tests reviewed
- [ ] Manual smoke test of affected features
- [ ] Regression test added for any new behavior
```

---

## Configuring AI Tools for Regression Safety

```markdown
# copilot-instructions.md
## Regression Prevention
- Run `npm test` after EVERY code change
- NEVER modify existing test files unless explicitly asked
- If tests fail after your change, fix the implementation
- Always check if changed functions are used elsewhere
- Add regression tests for any bug fixes
```

```markdown
# CLAUDE.md
## Testing Protocol
- After any code change: run full test suite
- Treat failing existing tests as bugs in YOUR code
- Do NOT update test expectations to match your implementation
- Before modifying any function, find all callers first
```
