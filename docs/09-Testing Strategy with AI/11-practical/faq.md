# Testing with AI FAQ

> Frequently asked questions about AI-assisted testing — practical answers for real-world development.

---

## General Questions

### Q: Should I let AI write all my tests?

**A: No.** AI generates test code, but you define what to test. The best workflow:

- **You** write the test specification (what behaviors to verify)
- **AI** generates the test code (the implementation of your specification)
- **You** review the tests for correctness and completeness
- **AI** can then help expand edge cases you identify

Think of AI as a fast typist who follows your testing blueprint.

---

### Q: How much test coverage is enough?

**A: 80% line coverage + high mutation score is a good target.**

| Metric | Target | Why |
|--------|--------|-----|
| Line coverage | 80%+ | Ensures most code is exercised |
| Branch coverage | 70%+ | Ensures conditional logic is tested |
| Mutation score | 70%+ | Ensures tests actually catch bugs |

Coverage alone is misleading — 100% line coverage with 20% mutation score means tests execute code but don't verify it. Always pair coverage with mutation testing.

---

### Q: Should AI run tests after generating code?

**A: Always — configure it in your instruction files.**

```markdown
# copilot-instructions.md
After every code change, run: npm test
Report the results. If tests fail, fix the code, not the tests.

# CLAUDE.md  
Run `npm test` after any code modification.
Never mark work complete without passing tests.
```

AI that doesn't run tests is like a developer who commits without building — technically possible but reckless.

---

### Q: Is TDD or test-after better with AI?

**A: TDD is dramatically better.** The data is clear:

| Approach | Bug Detection | Test Quality | Speed |
|----------|:------------:|:------------:|:-----:|
| **Test-first (TDD)** | High | High | Medium |
| **Test-after** | Low | Low | Fast (initially) |

With test-after, AI writes tests that mirror its own code (circular validation). With TDD, your tests constrain the AI and catch its mistakes. The initial slowdown pays back immediately in fewer bugs.

See: [Test-First Development with AI](../09-best-practices/test-first-development.md)

---

### Q: Can AI find bugs through testing?

**A: Yes, but you need to direct it.**

AI is good at:
- Generating edge case tests you didn't think of
- Finding boundary conditions through property-based testing
- Suggesting test scenarios for complex logic

AI is bad at:
- Adversarial testing (trying to break its own code)
- Security testing without explicit guidance
- Business logic validation (doesn't know your domain)

**Prompt pattern:**

```markdown
Review this function and suggest 5 ways it could fail that aren't
tested. Think adversarially — how would a malicious user break this?
```

---

## Test Generation Questions

### Q: AI generates tests that don't compile. How do I fix this?

**A: Provide more context in your prompts.** AI needs:

1. **Framework:** "Use Jest" or "Use Pytest"
2. **Import paths:** "Import from `../services/UserService`"
3. **Project structure:** "Tests go in `tests/` next to source files"
4. **Available libraries:** "We use @testing-library/react, not enzyme"

See: [Troubleshooting AI Testing](troubleshooting.md)

---

### Q: AI tests are too simple. How do I get better tests?

**A: Be specific about behaviors, not just functions.**

```markdown
# ❌ Vague: "Write tests for processOrder"
# ✅ Specific:
Write tests for processOrder that verify:
- Order with valid items succeeds
- Order with out-of-stock items returns error with item names
- Order total includes tax based on shipping state
- Discount codes reduce total (flat and percentage)
- Empty cart throws EmptyCartError
- Items with quantity > 100 require approval flag
```

---

### Q: How do I prevent AI from writing tests that mirror the implementation?

**A: Write tests before the implementation exists (TDD), or provide the specification instead of the code.**

```markdown
# ❌ "Write tests for this function" + [paste function]
# ✅ "Write tests for this specification:"
#    - Input: user object { name, email, age }
#    - Output: true if all fields valid, error object if not
#    - email must contain @
#    - age must be 13-120
#    - name must be 1-100 characters
```

See: [Anti-Pattern: Testing to Pass](../10-anti-patterns/testing-to-pass.md)

---

### Q: Should I use AI for E2E tests?

**A: Yes, but provide the user flow explicitly.**

AI is great at writing E2E test code once you describe the flow. It's terrible at knowing which flows matter.

```markdown
# You define the flow:
User flow: Checkout
1. User adds item to cart from /products page
2. User navigates to /cart
3. User clicks "Checkout"
4. User fills shipping form
5. User enters payment
6. User sees confirmation with order number

# AI writes the Playwright/Cypress code for this flow
```

---

## Tool-Specific Questions

### Q: How do I set up testing with GitHub Copilot?

**A:** Create `.github/copilot-instructions.md` with your testing conventions:

```markdown
## Testing
- Framework: Jest + React Testing Library
- Run tests: `npm test`
- Write tests BEFORE implementation
- Never modify existing tests for new code
- Minimum coverage: 80%
```

Copilot will follow these instructions across all chat sessions in the project.

---

### Q: How do I set up testing with Claude Code?

**A:** Create `CLAUDE.md` in your project root:

```markdown
## Testing
- Run all tests: `npm test`
- Run specific: `npm test -- --testPathPattern="auth"`
- Always run tests after code changes
- Read tests.json for current test status
- TDD workflow: tests first, then implement
```

Claude Code reads CLAUDE.md automatically at session start.

---

### Q: How do I track test state across AI sessions?

**A:** Use a `tests.json` file that persists test status as structured data:

```json
{
  "tests": [
    { "name": "User registration", "file": "tests/auth.test.js", "status": "pass" },
    { "name": "Password reset", "file": "tests/auth.test.js", "status": "pending" }
  ]
}
```

Tell AI to read this file at session start and update it after running tests.

See: [Structured Test Tracking](../09-best-practices/structured-test-tracking.md) and [Testing Across Sessions](../09-best-practices/testing-across-sessions.md)

---

## Test Quality Questions

### Q: How do I know if my tests actually catch bugs?

**A: Use mutation testing.** It introduces small changes (mutations) to your code and checks if your tests detect them.

```bash
# JavaScript
npx stryker run

# Python
pip install mutmut && mutmut run
```

A mutation score of 70%+ means your tests catch 70% of potential bugs. Below 50% means most tests are superficial.

See: [Mutation Testing](../05-coverage-and-quality/mutation-testing.md)

---

### Q: What's the difference between coverage and test quality?

**A:**

| Metric | Measures | Can Be Gamed? |
|--------|---------|:-------------:|
| **Line coverage** | How much code runs during tests | Yes — tests can run code without verifying it |
| **Branch coverage** | How many conditional paths are tested | Somewhat — can test paths without meaningful assertions |
| **Mutation score** | How many code changes are detected by tests | Hard to game — requires tests that actually verify behavior |
| **Assertion density** | How many checks per test | Somewhat — can add trivial assertions |

**Coverage tells you what's executed. Mutation testing tells you what's verified.**

---

### Q: My tests are flaky. How do I fix this?

**A:** AI-generated tests are especially prone to flakiness. Common causes and fixes:

| Cause | Fix |
|-------|-----|
| `setTimeout` in tests | Use `waitFor` or event-driven waiting |
| Shared state | Add `beforeEach` cleanup |
| Order dependency | Run with `--randomize` to detect |
| Hardcoded ports | Use dynamic port allocation |
| Date/time | Use fake timers (`jest.useFakeTimers()`) |

See: [Flaky Test Detection](../08-regression-and-maintenance/flaky-test-detection.md)

---

## Process Questions

### Q: How should I review AI-generated tests?

**A:** Apply the same rigor as production code review:

```markdown
## AI Test Review Checklist
- [ ] Tests verify behavior, not implementation
- [ ] Edge cases are covered
- [ ] Error handling is tested
- [ ] No over-mocking
- [ ] No snapshot abuse
- [ ] Naming is descriptive
- [ ] Tests can actually fail (they're not tautological)
```

---

### Q: When should I delete AI-generated tests?

**A:** Remove tests that:

- Verify deleted features
- Are permanently skipped with no fix plan
- Are exact duplicates of other tests
- Test implementation details rather than behavior
- Are tautological (can never fail)

Never delete a test just because it's failing — investigate first.

---

### Q: How do I handle AI that keeps modifying my tests?

**A:** Make it a hard rule in your instruction file:

```markdown
# CRITICAL RULE
You must NEVER modify files matching these patterns:
- tests/**/*.test.*
- **/*.spec.*
- **/__tests__/**

If these tests fail after your changes, fix your implementation.
The only exception is if I explicitly ask you to update a test.
```

---

### Q: Should I test AI-generated utility functions differently than AI-generated business logic?

**A: Yes.** Scale your testing rigor with risk:

| Code Type | Testing Rigor | Why |
|-----------|:------------:|-----|
| Utility functions | Standard unit tests | Low risk, well-defined behavior |
| Business logic | TDD + integration tests | High risk, domain-specific |
| Security code | TDD + property tests + review | Critical, adversarial environment |
| Data migrations | Full integration + backup tests | Irreversible, high impact |

---

### Q: How many tests should I have per feature?

**A:** A rough guideline:

```
Simple utility function:  3-5 tests
Service/business logic:   8-15 tests
API endpoint:            10-20 tests (unit + integration)
User flow (E2E):          3-5 scenarios
```

Quality matters more than quantity. 5 well-designed tests beat 50 superficial ones.
