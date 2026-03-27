# Test Coverage Gates

> Ensuring adequate test coverage for new and existing code.

---

## Why Coverage Gates Matter

Test coverage gates prevent untested code from reaching production. This is especially important for AI-generated code, which may appear correct but contain subtle logic errors only tests would catch.

---

## Coverage Metrics

| Metric | What It Measures | Typical Threshold |
|--------|-----------------|-------------------|
| **Line coverage** | % of code lines executed by tests | 80% for new code |
| **Branch coverage** | % of conditional branches tested | 70% for new code |
| **Function coverage** | % of functions called by tests | 90% for new code |
| **Statement coverage** | % of statements executed | 80% for new code |

---

## Implementation

### Jest (JavaScript/TypeScript)
```yaml
- name: Test with coverage
  run: |
    npx jest --coverage --coverageThreshold='{
      "global": {
        "branches": 70,
        "functions": 80,
        "lines": 80,
        "statements": 80
      }
    }'
```

### Pytest (Python)
```yaml
- name: Test with coverage
  run: |
    pytest --cov=src --cov-report=xml --cov-fail-under=80
```

---

## New Code vs Overall Coverage

Block on **new code coverage**, not overall:

```yaml
- name: Check new code coverage
  uses: codecov/codecov-action@v4
  with:
    flags: unittests
    fail_ci_if_error: true
    # Codecov can enforce coverage thresholds for changed files only
```

This avoids blocking PRs due to legacy untested code while ensuring all new code is properly tested.

---

## Coverage Anti-Patterns

| Anti-Pattern | Problem | Fix |
|-------------|---------|-----|
| **100% coverage target** | Tests become meaningless (assert true) | Set realistic thresholds (80%) |
| **Coverage without assertions** | Code runs but nothing is verified | Require meaningful assertions |
| **Testing implementation** | Tests break on refactoring | Test behavior, not implementation |
| **Ignoring edge cases** | Happy path only | Require branch coverage ≥70% |

---

## Key Takeaways

1. **80% line coverage for new code** — Realistic and effective threshold
2. **Gate new code, not overall** — Don't punish teams for legacy debt
3. **Branch coverage catches edge cases** — More valuable than line coverage alone
4. **Coverage ≠ quality** — High coverage with weak assertions is useless
5. **Ratchet up over time** — Start at 60%, increase to 80% over months

---

## Next Steps

- 🔗 [AI-Specific Quality Gates](ai-specific-quality-gates.md) — Special considerations for AI code
- 🔗 [Testing Strategies](../06-testing-ai-code/testing-strategies.md) — Comprehensive testing approaches
