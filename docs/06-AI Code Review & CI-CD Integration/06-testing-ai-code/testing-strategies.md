# Testing Strategies for AI-Generated Code

> Comprehensive approaches to validating code produced by AI agents and assistants.

---

## The Testing Pyramid for AI Code

```
        ╱╲
       ╱  ╲       Manual Exploratory Testing
      ╱    ╲      (Rare, for novel features)
     ╱──────╲
    ╱        ╲     E2E Tests
   ╱          ╲    (Critical user flows)
  ╱────────────╲
 ╱              ╲   Integration Tests
╱                ╲  (Module boundaries, API contracts)
╱──────────────────╲
╱                    ╲  Unit Tests
╱                      ╲ (Every function, every edge case)
```

---

## Strategy 1: Test Before You Trust

**Treat AI-generated code like a junior developer's PR.** It may look correct but:
- Logic might be subtly wrong
- Edge cases may be missing
- Error handling may be incomplete
- It may duplicate existing utilities

### The Checklist
1. ✅ Read the code line-by-line (don't just scan)
2. ✅ Verify it solves the **right** problem
3. ✅ Check for missing error handling
4. ✅ Look for hardcoded values that should be configurable
5. ✅ Verify imports exist and are current
6. ✅ Run existing tests (does it break anything?)
7. ✅ Write new tests for the new code

---

## Strategy 2: AI Writes Code, Human Writes Tests

The safest pattern: AI generates the implementation, humans write the tests.

```
1. AI generates function
2. Human reviews function
3. Human writes tests (specifying expected behavior)
4. Tests verify AI code is correct
5. Tests serve as documentation of intent
```

**Why**: Tests written by humans encode the **human's understanding** of what the code should do, catching cases where AI misunderstood the requirement.

---

## Strategy 3: AI Writes Both (with Validation)

If AI writes both code and tests, add extra validation:

1. **Mutation testing**: Deliberately break the code; tests should fail
2. **Property-based testing**: Random inputs to find edge cases
3. **Coverage analysis**: Ensure tests actually exercise the code paths
4. **Human review of tests**: Verify tests are meaningful, not just "assert true"

---

## Strategy 4: Test-First with AI Implementation

The TDD approach works well with AI:

```
1. Human writes failing tests first (defines the spec)
2. AI implements code to make tests pass
3. Human reviews the implementation
4. Refactor if needed
```

**Advantage**: Tests define the contract; AI can't deviate from the spec.

---

## Testing Types for AI Code

| Test Type | Why It Matters for AI Code |
|-----------|--------------------------|
| **Unit tests** | Catches logic errors in individual functions |
| **Integration tests** | Verifies AI code works with existing modules |
| **E2E tests** | Ensures user workflows still work |
| **Property-based tests** | Finds edge cases AI missed |
| **Mutation tests** | Verifies tests actually catch bugs |
| **Security tests** | Catches insecure patterns from training data |
| **Performance tests** | AI code can be inefficient |
| **Snapshot tests** | Detects unexpected output changes |

---

## Key Takeaways

1. **Never trust AI code without testing** — Treat it like a junior dev's work
2. **Human writes tests, AI writes code** — Safest pattern
3. **TDD works great with AI** — Tests define the spec, AI implements
4. **Mutation testing catches weak tests** — Especially for AI-generated tests
5. **Test all levels** — Unit + integration + E2E + security

---

## Next Steps

- 🔗 [AI Test Generation](ai-test-generation.md) — Using AI to write tests
- 🔗 [Validation Framework](validation-framework.md) — Structured validation pipeline
