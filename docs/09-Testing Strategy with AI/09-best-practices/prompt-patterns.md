# Test Prompt Patterns

> A library of effective prompts for AI test generation — each pattern with a template, concrete example, and guidance on when to use it.

---

## Why Prompt Patterns Matter

The quality of AI-generated tests depends directly on how you ask for them. Vague prompts produce vague tests. Structured prompts produce thorough, meaningful tests.

```
❌ "Write tests for this function"
   → AI copies function logic into test assertions

✅ "Write tests for this function that verify [specific behaviors]
    including edge cases for [boundaries] and error handling for [failures]"
   → AI produces tests that actually catch bugs
```

---

## Pattern 1: Unit Test Generation

**When to use:** Testing individual functions or methods in isolation.

### Template

```markdown
Generate unit tests for the `[functionName]` function in `[filePath]`.

Test these behaviors:
- [behavior 1 — normal case]
- [behavior 2 — edge case]
- [behavior 3 — error case]

Use [framework] (e.g., Jest/Pytest/JUnit).
Each test should have a descriptive name explaining the scenario.
Do NOT mock the function itself — test its real behavior.
```

### Example

```markdown
Generate unit tests for the `calculateShipping` function in `src/shipping.js`.

Test these behaviors:
- Standard shipping for orders under $50 costs $5.99
- Free shipping for orders $50 and above
- Express shipping adds $10 to any order
- International shipping doubles the base rate
- Throws error for negative order amounts
- Throws error for unsupported shipping methods
- Handles zero-dollar orders (free items)

Use Jest. Each test should have a descriptive name.
```

### Expected Output Quality

```javascript
describe('calculateShipping', () => {
  test('charges $5.99 for standard shipping under $50', () => {
    expect(calculateShipping(30, 'standard')).toBe(5.99);
  });

  test('provides free standard shipping for orders $50 and above', () => {
    expect(calculateShipping(50, 'standard')).toBe(0);
    expect(calculateShipping(100, 'standard')).toBe(0);
  });

  test('throws error for negative order amounts', () => {
    expect(() => calculateShipping(-10, 'standard')).toThrow('Invalid order amount');
  });
  // ...
});
```

---

## Pattern 2: Edge Case Discovery

**When to use:** After basic tests exist and you want to find gaps.

### Template

```markdown
Review the `[functionName]` function and its existing tests.

Function: [paste function or reference file]
Existing tests: [paste tests or reference file]

Identify edge cases NOT covered by existing tests. Consider:
- Boundary values (0, -1, MAX_INT, empty strings)
- Null/undefined inputs
- Type coercion issues
- Concurrent access
- Unicode/special characters
- Very large inputs
- State transitions

Generate tests for the top 5 most important uncovered edge cases.
```

### Example

```markdown
Review the `parseCSV` function and its existing tests.

Function: src/utils/csv-parser.js
Existing tests: tests/csv-parser.test.js

Identify edge cases NOT covered. Consider:
- Empty files, files with only headers
- Fields containing commas, quotes, newlines
- Different line endings (CRLF vs LF)
- Very large files (memory concerns)
- Malformed CSV (unmatched quotes)
- Unicode characters in fields
- Trailing commas, leading spaces

Generate tests for the top 5 most critical uncovered cases.
```

---

## Pattern 3: Integration Test Generation

**When to use:** Testing how multiple components work together.

### Template

```markdown
Write integration tests for the `[API endpoint / service interaction]`.

Components involved:
- [Component 1] — [its role]
- [Component 2] — [its role]
- [Component 3] — [its role]

Test these integration scenarios:
- [Happy path through all components]
- [Failure in component 2 — how does the system respond?]
- [Timeout/slow response handling]
- [Data consistency across components]

Use a real [database/service] (not mocked) for [specific component].
Mock only [external services that can't be tested locally].
```

### Example

```markdown
Write integration tests for the POST /api/orders endpoint.

Components involved:
- OrderController — validates request, returns response
- OrderService — business logic, calculates totals
- OrderRepository — saves to PostgreSQL
- PaymentClient — calls Stripe API

Test these integration scenarios:
- Successful order creation (validates → calculates → saves → charges)
- Invalid order data returns 400 with validation errors
- Database save failure returns 500 and doesn't charge payment
- Payment failure returns 402 and doesn't save order
- Duplicate order detection returns 409

Use a real test database for OrderRepository.
Mock only PaymentClient (Stripe).
```

---

## Pattern 4: Property-Based Tests

**When to use:** Testing pure functions where properties should hold for ALL inputs.

### Template

```markdown
Write property-based tests for `[functionName]` using [fast-check / Hypothesis / jqwik].

Properties to verify:
- [Invariant 1 — something always true about the output]
- [Invariant 2 — relationship between input and output]
- [Round-trip property — encode then decode returns original]
- [Idempotency — applying twice equals applying once]

Generate appropriate input arbitraries for [input types].
```

### Example

```markdown
Write property-based tests for `sortUsers` using fast-check.

Properties to verify:
- Output length equals input length (no items lost or duplicated)
- Output is actually sorted by the specified field
- Sorting is stable (equal elements maintain relative order)
- Sorting the already-sorted output produces the same result (idempotent)
- Every element in the output exists in the input

Generate arbitraries for arrays of user objects with name (string) and age (number).
```

### Expected Output

```javascript
import fc from 'fast-check';

const userArbitrary = fc.record({
  name: fc.string({ minLength: 1 }),
  age: fc.integer({ min: 0, max: 150 }),
});

describe('sortUsers property-based tests', () => {
  test('preserves array length', () => {
    fc.assert(
      fc.property(fc.array(userArbitrary), (users) => {
        const sorted = sortUsers(users, 'name');
        expect(sorted).toHaveLength(users.length);
      })
    );
  });

  test('output is sorted by specified field', () => {
    fc.assert(
      fc.property(fc.array(userArbitrary), (users) => {
        const sorted = sortUsers(users, 'age');
        for (let i = 1; i < sorted.length; i++) {
          expect(sorted[i].age).toBeGreaterThanOrEqual(sorted[i - 1].age);
        }
      })
    );
  });
});
```

---

## Pattern 5: Regression Tests for Bugs

**When to use:** A bug was found — write a test that fails with the bug and passes with the fix.

### Template

```markdown
Write a regression test for this bug:

**Bug:** [describe the incorrect behavior]
**Expected:** [what should happen]
**Actual:** [what happens instead]
**Reproduction:** [steps or input that triggers it]

The test should:
1. FAIL with the current (buggy) code
2. PASS after the bug is fixed
3. Prevent this specific bug from recurring

Include a comment referencing the bug/issue number.
```

### Example

```markdown
Write a regression test for this bug:

**Bug:** User discount is applied twice when coupon code contains uppercase letters
**Expected:** 20% discount applied once → $80 total for $100 order
**Actual:** 20% applied twice → $64 total for $100 order
**Reproduction:** Apply coupon "SAVE20" (uppercase) to a $100 order

The test should fail with the current code and pass after the fix.
Reference issue #287.
```

### Expected Output

```javascript
// Regression test for #287: double discount with uppercase coupons
test('applies discount only once regardless of coupon case (#287)', () => {
  const order = createOrder({ subtotal: 100 });

  const result = applyCoupon(order, 'SAVE20');

  expect(result.total).toBe(80);          // 20% off once = $80
  expect(result.discountApplied).toBe(20); // $20 discount, not $36
});
```

---

## Pattern 6: E2E Tests for User Flows

**When to use:** Testing complete user workflows through the application.

### Template

```markdown
Write E2E tests for the [user flow name] using [Playwright / Cypress].

User flow steps:
1. [User action 1]
2. [Expected result 1]
3. [User action 2]
4. [Expected result 2]
...

Test variations:
- Happy path (everything succeeds)
- [Failure scenario 1]
- [Failure scenario 2]

Use data-testid attributes for selectors. Include proper waits for async operations.
Do NOT use arbitrary timeouts — use `waitFor` or element visibility.
```

### Example

```markdown
Write E2E tests for the checkout flow using Playwright.

User flow:
1. User navigates to /cart with items
2. User clicks "Proceed to Checkout"
3. User fills shipping form (name, address, city, zip)
4. User selects shipping method (standard/express)
5. User enters payment details
6. User clicks "Place Order"
7. User sees order confirmation with order number

Test variations:
- Happy path with standard shipping
- Happy path with express shipping
- Invalid shipping address shows error
- Declined payment shows error and returns to payment step
- Empty cart redirects to /shop

Use data-testid attributes. Use proper waits.
```

---

## Prompt Quality Checklist

Before sending any test generation prompt to AI, verify:

```markdown
## Prompt Quality Checklist
- [ ] **Specific behaviors listed** — not just "write tests for this"
- [ ] **Edge cases mentioned** — boundaries, nulls, empty inputs
- [ ] **Error cases included** — what should fail and how
- [ ] **Framework specified** — Jest, Pytest, JUnit, etc.
- [ ] **Mocking boundaries clear** — what to mock, what to test real
- [ ] **Naming convention stated** — descriptive test names expected
- [ ] **Context provided** — function signature, types, dependencies
- [ ] **Anti-patterns prohibited** — "don't copy implementation logic"
```

---

## Prompt Anti-Patterns

| ❌ Bad Prompt | Why It Fails | ✅ Better Prompt |
|--------------|-------------|-----------------|
| "Write tests" | No context, no behaviors | "Write tests that verify [behaviors]" |
| "Write tests for this code" | AI mirrors the code | "Write tests from the specification: [spec]" |
| "Get to 100% coverage" | Generates meaningless tests | "Cover these critical paths: [list]" |
| "Add more tests" | No direction | "Add edge case tests for [boundaries]" |
| "Make sure it works" | Not actionable | "Verify that [specific scenario] produces [result]" |

---

## Quick Reference Card

| Pattern | Template Start | Best For |
|---------|---------------|----------|
| Unit test | "Generate unit tests for `[fn]` that verify..." | Pure functions, utilities |
| Edge cases | "Identify untested edge cases for..." | Coverage gaps |
| Integration | "Write integration tests for `[endpoint]`..." | API routes, service chains |
| Property-based | "Write property-based tests verifying..." | Pure functions, serialization |
| Regression | "Write a regression test for bug: [description]" | Bug fixes |
| E2E | "Write E2E tests for [user flow]..." | User journeys |

---

## Next Steps

- [Test-First Development with AI](test-first-development.md) — the workflow these prompts support
- [AI Test Generation Patterns](../03-test-generation/generation-patterns.md) — broader generation strategies
- [Edge Case Discovery](../03-test-generation/edge-case-discovery.md) — finding what's missing
- [Testing Templates](../11-practical/templates.md) — ready-to-use templates for these patterns

---

*Part of the [Testing Strategy with AI](../README.md) series.*
