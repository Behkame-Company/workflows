# Test as Specification

> Tests are the only specifications that machines can verify. Write the test, let AI write the implementation.

---

## Why Tests Beat Prose as Specifications

When you give AI a natural language description, you get **one possible interpretation**. When you give AI a test suite, you get **the only implementation that passes**.

### The Ambiguity Problem

```
Prose Spec:
  "The function should handle large inputs efficiently
   and return appropriate error messages for invalid data."

Questions AI must guess at:
  - How large is "large"? (100? 10,000? 1,000,000?)
  - What's "efficiently"? (< 1 second? < 100ms? O(n)?)
  - What's "invalid data"? (null? empty? wrong type? negative?)
  - What's "appropriate"? (throw? return null? return error object?)
```

Every ambiguity is a potential bug. AI will make reasonable-sounding but potentially wrong guesses for every one.

### The Test Spec Solution

```python
import pytest

def test_handles_large_input():
    """Processes 100,000 items in under 500ms"""
    items = list(range(100_000))
    import time
    start = time.time()
    result = process_items(items)
    elapsed = (time.time() - start) * 1000
    assert elapsed < 500
    assert len(result) == 100_000

def test_rejects_none_input():
    with pytest.raises(ValueError, match="Input cannot be None"):
        process_items(None)

def test_rejects_empty_input():
    with pytest.raises(ValueError, match="Input cannot be empty"):
        process_items([])

def test_rejects_non_numeric_items():
    with pytest.raises(TypeError, match="All items must be numeric"):
        process_items([1, "two", 3])
```

**Zero ambiguity.** The AI now knows exactly what "large" means, what "efficient" means, what "invalid" means, and what "appropriate" means.

---

## The Tests-as-Specs Pattern

### Workflow

```
┌─────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│  1. Human writes │     │  2. AI writes     │     │  3. Tests verify │
│     tests first  │ ──▶ │     implementation │ ──▶ │     correctness  │
│  (specification) │     │  (to pass tests)  │     │  (automated)     │
└─────────────────┘     └──────────────────┘     └─────────────────┘
         │                                                 │
         │              ┌──────────────────┐               │
         └───────────── │  4. Refine until  │ ◀────────────┘
                        │     all tests pass │
                        └──────────────────┘
```

### Step-by-Step

**Step 1: Write the test file**

```typescript
// user-validator.test.ts — This IS the specification
import { validateUser } from './user-validator';

describe('validateUser', () => {
  test('accepts valid user with all required fields', () => {
    const result = validateUser({
      name: 'Alice Johnson',
      email: 'alice@example.com',
      age: 30,
    });
    expect(result.valid).toBe(true);
    expect(result.errors).toEqual([]);
  });

  test('rejects user without name', () => {
    const result = validateUser({ email: 'a@b.com', age: 25 });
    expect(result.valid).toBe(false);
    expect(result.errors).toContain('Name is required');
  });

  test('rejects invalid email format', () => {
    const result = validateUser({ name: 'Bob', email: 'not-an-email', age: 25 });
    expect(result.valid).toBe(false);
    expect(result.errors).toContain('Email must be a valid email address');
  });

  test('rejects age under 13', () => {
    const result = validateUser({ name: 'Kid', email: 'k@b.com', age: 12 });
    expect(result.valid).toBe(false);
    expect(result.errors).toContain('User must be at least 13 years old');
  });

  test('rejects age over 150', () => {
    const result = validateUser({ name: 'Old', email: 'o@b.com', age: 151 });
    expect(result.valid).toBe(false);
    expect(result.errors).toContain('Age must be realistic (under 150)');
  });

  test('collects multiple errors', () => {
    const result = validateUser({ email: 'bad', age: 5 });
    expect(result.valid).toBe(false);
    expect(result.errors).toHaveLength(3); // name, email, age
  });
});
```

**Step 2: Prompt AI with the test file**

```
Here are my tests in user-validator.test.ts.
Write the implementation in user-validator.ts that makes all tests pass.
```

**Step 3: Run tests, iterate if needed**

```bash
$ npx vitest run user-validator.test.ts
 ✓ accepts valid user with all required fields
 ✓ rejects user without name
 ✓ rejects invalid email format
 ✓ rejects age under 13
 ✓ rejects age over 150
 ✓ collects multiple errors

 6 passed
```

---

## Structured Test Tracking with tests.json

For multi-session work and complex features, a `tests.json` file tracks test status across sessions. This format is recommended by Anthropic for AI coding agents:

### The tests.json Format

```json
{
  "feature": "User Authentication",
  "created": "2024-01-15",
  "updated": "2024-01-16",
  "summary": {
    "total": 12,
    "passed": 8,
    "failed": 2,
    "pending": 2
  },
  "tests": [
    {
      "id": "auth-001",
      "name": "Valid login returns JWT token",
      "file": "src/auth/__tests__/login.test.ts",
      "status": "passed",
      "category": "unit"
    },
    {
      "id": "auth-002",
      "name": "Invalid password returns 401",
      "file": "src/auth/__tests__/login.test.ts",
      "status": "passed",
      "category": "unit"
    },
    {
      "id": "auth-003",
      "name": "Expired token triggers refresh flow",
      "file": "src/auth/__tests__/token.test.ts",
      "status": "failed",
      "error": "Token refresh not yet implemented",
      "category": "integration"
    },
    {
      "id": "auth-004",
      "name": "Rate limiting after 5 failed attempts",
      "file": "src/auth/__tests__/security.test.ts",
      "status": "pending",
      "category": "integration"
    },
    {
      "id": "auth-005",
      "name": "Full login-to-dashboard flow",
      "file": "e2e/auth.spec.ts",
      "status": "pending",
      "category": "e2e"
    }
  ]
}
```

### Why tests.json Matters for AI Workflows

```
Session 1: Write tests for auth-001 through auth-005
           → Update tests.json status
           → Commit

Session 2: AI reads tests.json, sees auth-003 failed
           → Focuses on token refresh implementation
           → Runs tests, updates status
           → Commit

Session 3: AI reads tests.json, sees auth-004 and auth-005 pending
           → Implements rate limiting
           → Writes E2E test
           → Updates status
           → Commit
```

The `tests.json` provides **continuity across context resets** — each new session immediately knows what's done, what's failing, and what's pending.

### Prompting AI with tests.json

```
Read the tests.json file. Focus on tests with status "failed" or "pending".
For each failed test, fix the implementation to make it pass.
For each pending test, implement the test and the supporting code.
Update tests.json with new statuses after running the test suite.
```

---

## Before and After: Prose Spec vs Test Spec

### ❌ Before: Prose Specification

```markdown
## Shopping Cart Requirements

The shopping cart should:
- Allow adding items with quantities
- Calculate totals correctly
- Apply discount codes
- Handle tax calculations based on region
- Not allow negative quantities
- Support removing items
- Clear the cart after checkout
```

**Problems**: What does "correctly" mean? How are discounts applied — percentage or fixed? Which regions? What happens when removing an item that doesn't exist?

### ✅ After: Test Specification

```python
class TestShoppingCart:
    def test_add_item_with_quantity(self):
        cart = ShoppingCart()
        cart.add("widget", price=9.99, quantity=3)
        assert cart.item_count == 3
        assert cart.items["widget"].quantity == 3

    def test_total_calculation(self):
        cart = ShoppingCart()
        cart.add("widget", price=9.99, quantity=2)
        cart.add("gadget", price=24.99, quantity=1)
        assert cart.subtotal == 44.97  # (9.99 * 2) + 24.99

    def test_percentage_discount_code(self):
        cart = ShoppingCart()
        cart.add("widget", price=100.00, quantity=1)
        cart.apply_discount("SAVE20")  # 20% off
        assert cart.subtotal == 100.00  # subtotal unchanged
        assert cart.total == 80.00     # total reflects discount

    def test_fixed_discount_code(self):
        cart = ShoppingCart()
        cart.add("widget", price=100.00, quantity=1)
        cart.apply_discount("TAKE10")  # $10 off
        assert cart.total == 90.00

    def test_tax_calculation_california(self):
        cart = ShoppingCart(region="CA")
        cart.add("widget", price=100.00, quantity=1)
        assert cart.tax == 7.25  # CA tax rate 7.25%
        assert cart.total == 107.25

    def test_tax_calculation_oregon(self):
        cart = ShoppingCart(region="OR")
        cart.add("widget", price=100.00, quantity=1)
        assert cart.tax == 0.00  # OR has no sales tax
        assert cart.total == 100.00

    def test_reject_negative_quantity(self):
        cart = ShoppingCart()
        with pytest.raises(ValueError, match="Quantity must be positive"):
            cart.add("widget", price=9.99, quantity=-1)

    def test_remove_existing_item(self):
        cart = ShoppingCart()
        cart.add("widget", price=9.99, quantity=2)
        cart.remove("widget")
        assert cart.item_count == 0

    def test_remove_nonexistent_item_raises(self):
        cart = ShoppingCart()
        with pytest.raises(KeyError, match="Item 'widget' not in cart"):
            cart.remove("widget")

    def test_clear_after_checkout(self):
        cart = ShoppingCart()
        cart.add("widget", price=9.99, quantity=1)
        cart.checkout(payment_method="card")
        assert cart.item_count == 0
        assert cart.total == 0.00
```

**Every ambiguity resolved.** Every edge case defined. AI can implement this with confidence — and you can verify with certainty.

---

## Writing Effective Test Specifications

### Principles

1. **Start with the interface** — Define the function/class signature through how tests call it
2. **Cover the happy path first** — Then systematically add edge cases
3. **Make error behavior explicit** — Specify exact error types and messages
4. **Include boundary values** — Zero, one, max, and just-beyond limits
5. **Define the return shape** — Tests show exactly what structure to expect

### Template for Test-as-Spec

```javascript
describe('FeatureName', () => {
  // Happy path — the core behavior
  test('does the main thing correctly', () => { /* ... */ });

  // Input boundaries
  test('handles minimum valid input', () => { /* ... */ });
  test('handles maximum valid input', () => { /* ... */ });

  // Error cases
  test('rejects invalid input with specific error', () => { /* ... */ });
  test('handles null/undefined gracefully', () => { /* ... */ });

  // Edge cases
  test('handles empty collections', () => { /* ... */ });
  test('handles duplicate entries', () => { /* ... */ });
  test('handles concurrent access', () => { /* ... */ });

  // Integration points
  test('works with dependency X', () => { /* ... */ });
  test('handles dependency failure', () => { /* ... */ });
});
```

---

## Practical Tips

### ✅ Do

- **Write tests before prompting AI** for implementation
- **Use tests.json** to track progress across sessions
- **Include exact expected values** in assertions — no vague checks
- **Define error messages in tests** — AI will use them in the implementation
- **Version your test specs** — they're documentation, not throwaway code

### ❌ Don't

- **Don't describe behavior in comments** when you can express it in assertions
- **Don't write tests after AI implements** — that's verification, not specification
- **Don't use vague assertions** like `toBeTruthy()` when you mean `toBe(true)`
- **Don't let AI write the spec tests** — that defeats the purpose of independent specification

---

## Key Takeaways

> **Tests are the specification language that AI can actually execute.** Prose describes intent; tests define behavior.

1. Tests eliminate ambiguity that prose specifications leave open
2. The tests-as-specs workflow: write tests → AI implements → verify → iterate
3. `tests.json` provides continuity across AI sessions and context resets
4. Every assertion is a requirement; every test is a user story
5. Human writes the spec (tests), AI writes the implementation — not the other way around
