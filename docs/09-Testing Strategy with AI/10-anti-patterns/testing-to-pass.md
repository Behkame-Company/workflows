# Anti-Pattern: Testing to Pass

> When AI writes tests designed to pass rather than to verify — producing 100% pass rates with 0% bug-detection value.

---

## The Problem

When you ask AI to "write tests for this function," it reads the function's implementation and generates tests that mirror its logic. The tests don't verify correctness against a specification — they verify that the code does what the code does.

```
What you expect:                    What actually happens:
────────────────                    ────────────────────
Specification → Tests → Code        Code → Tests (that match code)
  (tests verify spec)                (tests verify code does what code does)
```

This creates a **circular validation loop**: the tests can only fail if the code changes, never if the code is wrong.

---

## How It Happens

### Step 1: You Ask AI to Generate Tests

```markdown
"Write unit tests for the calculateDiscount function in src/pricing.js"
```

### Step 2: AI Reads the Implementation

```javascript
// src/pricing.js
function calculateDiscount(price, customerType) {
  if (customerType === 'premium') {
    return price * 0.8;  // BUG: should be 0.85 (15% discount, not 20%)
  }
  return price;
}
```

### Step 3: AI Generates Tests That Match the Bug

```javascript
// ❌ AI-generated: tests the implementation, not the specification
test('applies premium discount', () => {
  expect(calculateDiscount(100, 'premium')).toBe(80);  // Matches the bug!
});

test('no discount for regular customers', () => {
  expect(calculateDiscount(100, 'regular')).toBe(100);
});

// Result: 2/2 tests pass ✅
// Reality: The premium discount is WRONG (20% instead of 15%)
// These tests will never catch this bug
```

---

## Detection: How to Spot Testing-to-Pass

### Signal 1: Tests Mirror Implementation Logic

```javascript
// Implementation:
function formatName(first, last) {
  return `${last}, ${first}`;
}

// ❌ Test mirrors implementation:
test('formats name', () => {
  expect(formatName('John', 'Doe')).toBe(`${'Doe'}, ${'John'}`);
  // This is literally the same logic restated
});

// ✅ Test from specification:
test('formats name as "Last, First"', () => {
  expect(formatName('John', 'Doe')).toBe('Doe, John');
  // Tests the expected OUTPUT, not the logic
});
```

### Signal 2: 100% Pass Rate on First Run

If AI writes 20 tests and every single one passes on the first run, be suspicious. Real specification-based tests typically surface at least one issue.

### Signal 3: Tests That Can't Catch Bugs

Ask yourself: "If I introduced a bug in this function, would these tests catch it?"

```javascript
// Implementation with a subtle bug
function isEligible(age, hasLicense) {
  return age > 16 && hasLicense;  // BUG: should be age >= 16
}

// ❌ AI-generated test (misses boundary):
test('eligible with valid age and license', () => {
  expect(isEligible(25, true)).toBe(true);   // Passes
  expect(isEligible(10, true)).toBe(false);   // Passes
});
// Never tests age === 16, so the >= vs > bug is invisible

// ✅ Specification-based test (catches boundary):
test('16 year old with license is eligible', () => {
  expect(isEligible(16, true)).toBe(true);  // FAILS — catches the bug!
});
```

### Signal 4: No Edge Cases or Error Handling

AI tests-to-pass typically test 2-3 happy paths and skip:
- Boundary values
- Null/undefined inputs
- Error conditions
- Concurrent scenarios

---

## Root Cause Analysis

| Root Cause | Description |
|-----------|-------------|
| **Prompt is code-centric** | "Write tests for this function" invites implementation mirroring |
| **No specification provided** | AI has nothing to test against except the code |
| **AI optimization bias** | AI is trained to produce "correct" output — passing tests feel correct |
| **Lack of adversarial thinking** | AI doesn't naturally try to break the code |

---

## The Fix: Specification-First Testing

### Fix 1: Write Tests BEFORE Code Exists

```markdown
# Prompt (before implementation):
Write tests for a calculateDiscount function that:
- Returns the original price for regular customers
- Applies 15% discount for premium customers
- Applies 25% discount for VIP customers
- Throws an error for negative prices
- Throws an error for unknown customer types
```

The tests are written against the **specification**, not the implementation.

### Fix 2: Provide the Specification, Not the Code

```markdown
# ❌ Bad: "Write tests for this function" + paste code
# ✅ Good: "Write tests for this specification"

Specification:
- calculateDiscount(price, customerType) → number
- Regular customers: no discount (return price)
- Premium customers: 15% discount
- VIP customers: 25% discount
- Invalid price (negative/NaN): throw "Invalid price"
- Unknown customer type: throw "Unknown customer type"
```

### Fix 3: Use Mutation Testing

Mutation testing changes the code and checks if tests detect the change:

```bash
# Install Stryker (JavaScript)
npm install --save-dev @stryker-mutator/core @stryker-mutator/jest-runner

# Run mutation testing
npx stryker run

# Output:
# Mutant: replaced > with >= in isEligible
# Status: SURVIVED ← Test didn't catch this mutation!
# Your tests are testing-to-pass.
```

```
Mutation Testing Results:
─────────────────────────
Total mutants:    15
Killed:            8  (tests caught the change)
Survived:          7  (tests MISSED the change) ← These are testing-to-pass
Mutation score:   53% (target: 80%+)
```

### Fix 4: Adversarial Prompting

```markdown
Write tests that would BREAK if any of these bugs were introduced:
1. Discount percentage is wrong (e.g., 20% instead of 15%)
2. Discount is applied to wrong customer type
3. Negative prices are accepted
4. The function returns undefined instead of throwing

Make each test verify one specific behavior. If a test would still
pass with one of these bugs, the test is too weak.
```

---

## Testing-to-Pass vs. Testing-to-Verify

| Aspect | Testing-to-Pass ❌ | Testing-to-Verify ✅ |
|--------|-------------------|---------------------|
| Source of truth | Implementation | Specification |
| When written | After code | Before code |
| Pass rate (initial) | 100% | ~70-90% (reveals issues) |
| Bug detection | Near zero | High |
| Mutation score | Low (~30-50%) | High (~80%+) |
| Value over time | Decreasing | Increasing |

---

## Prevention Checklist

```markdown
## PR Review: Testing-to-Pass Detection
- [ ] Tests were written before or independently of implementation
- [ ] Tests reference specification/requirements, not code
- [ ] At least one boundary value test exists per function
- [ ] Error cases are tested
- [ ] Tests don't replicate implementation logic in assertions
- [ ] Mutation testing score is above 80%
- [ ] At least one test failed during development (proves tests check something)
```

---

## Configuring AI to Avoid This Pattern

```markdown
# copilot-instructions.md or CLAUDE.md

## Test Quality Rules
- Write tests from SPECIFICATIONS, not from reading the code
- If no specification exists, ask for one before generating tests
- Every test must be able to FAIL — design tests that would catch bugs
- Include boundary values: 0, -1, MAX, empty, null
- Include error scenarios, not just happy paths
- After generating tests, explain what bug each test would catch
- Never generate tests by reading the implementation and asserting its output
```

---

## Next Steps

- [Test-First Development with AI](../09-best-practices/test-first-development.md) — the primary fix for testing-to-pass
- [Mutation Testing](../05-coverage-and-quality/mutation-testing.md) — automated detection of weak tests
- [Anti-Pattern: Over-Mocking](mock-everything.md) — another way tests verify nothing
- [Test Quality Metrics](../05-coverage-and-quality/test-quality-metrics.md) — measuring test effectiveness

---

*Part of the [Testing Strategy with AI](../README.md) series.*
