# TDD Workflow with AI Agents

> The Red-Green-Refactor cycle adapted for AI — where humans write the specification and AI fills the implementation.

---

## Why TDD + AI Is a Force Multiplier

Traditional TDD already separates **what** (the test) from **how** (the implementation). AI agents excel at the "how" — generating code to satisfy constraints — while humans excel at the "what" — defining correct behavior. TDD with AI formalizes this division:

| Role | Responsibility |
|------|---------------|
| **Human** | Writes failing tests (specification) |
| **AI** | Implements code to pass tests (execution) |
| **Both** | Refactor for quality and maintainability |

**Key benefit:** Tests verify the AI didn't hallucinate. If the test passes, the implementation is correct — regardless of whether the AI's approach matches what you'd have written.

---

## The AI-Enhanced TDD Loop

```
┌─────────────────────────────────────────────────────────┐
│                   AI-Enhanced TDD Cycle                  │
│                                                         │
│   ┌──────────┐    ┌──────────┐    ┌──────────────┐     │
│   │   RED    │───▶│  GREEN   │───▶│   REFACTOR   │     │
│   │  Human   │    │    AI    │    │  Human + AI   │     │
│   │  writes  │    │  writes  │    │  collaborate  │     │
│   │  failing │    │  minimum │    │  on clean     │     │
│   │  test    │    │  passing │    │  code         │     │
│   └──────────┘    │  code    │    └──────┬───────┘     │
│        ▲          └──────────┘           │              │
│        │                                 │              │
│        └─────────────────────────────────┘              │
│                  Next test case                         │
└─────────────────────────────────────────────────────────┘
```

### Step 1: RED — Human Writes a Failing Test

The human defines the expected behavior. This is the **specification** that constrains the AI.

```typescript
// Step 1: Human writes the test — this is the contract
describe('PasswordValidator', () => {
  it('should reject passwords shorter than 8 characters', () => {
    const validator = new PasswordValidator();
    expect(validator.validate('short')).toEqual({
      valid: false,
      errors: ['Password must be at least 8 characters'],
    });
  });

  it('should reject passwords without uppercase letters', () => {
    const validator = new PasswordValidator();
    expect(validator.validate('alllowercase1!')).toEqual({
      valid: false,
      errors: ['Password must contain at least one uppercase letter'],
    });
  });

  it('should accept valid passwords', () => {
    const validator = new PasswordValidator();
    expect(validator.validate('SecurePass1!')).toEqual({
      valid: true,
      errors: [],
    });
  });
});
```

**Run the tests — they must fail.** If they pass without implementation, your tests are wrong.

```bash
npx jest password-validator.test.ts
# FAIL: PasswordValidator is not defined
```

### Step 2: GREEN — AI Implements Minimum Passing Code

Give the AI your failing test file and ask it to implement the code:

**Prompt:**
```
Here are my failing tests for a PasswordValidator class.
Implement the minimum code needed to make all tests pass.
Do not add functionality beyond what the tests require.

[paste test file]
```

**AI generates:**

```typescript
// password-validator.ts — AI-generated implementation
interface ValidationResult {
  valid: boolean;
  errors: string[];
}

export class PasswordValidator {
  validate(password: string): ValidationResult {
    const errors: string[] = [];

    if (password.length < 8) {
      errors.push('Password must be at least 8 characters');
    }

    if (!/[A-Z]/.test(password)) {
      errors.push('Password must contain at least one uppercase letter');
    }

    return {
      valid: errors.length === 0,
      errors,
    };
  }
}
```

**Run the tests — they must pass.**

```bash
npx jest password-validator.test.ts
# PASS: 3/3 tests passing ✓
```

### Step 3: REFACTOR — Human + AI Collaborate

Now both human and AI can improve the code while tests protect against regressions:

**Prompt:**
```
The tests pass. Now refactor this PasswordValidator to:
1. Make validation rules configurable
2. Support custom error messages
3. Keep all existing tests passing
```

```typescript
// Refactored — AI improves structure while tests guard behavior
interface ValidationRule {
  test: (password: string) => boolean;
  message: string;
}

const DEFAULT_RULES: ValidationRule[] = [
  {
    test: (pw) => pw.length >= 8,
    message: 'Password must be at least 8 characters',
  },
  {
    test: (pw) => /[A-Z]/.test(pw),
    message: 'Password must contain at least one uppercase letter',
  },
];

export class PasswordValidator {
  constructor(private rules: ValidationRule[] = DEFAULT_RULES) {}

  validate(password: string): ValidationResult {
    const errors = this.rules
      .filter((rule) => !rule.test(password))
      .map((rule) => rule.message);

    return { valid: errors.length === 0, errors };
  }
}
```

```bash
npx jest password-validator.test.ts
# PASS: 3/3 tests still passing ✓ — refactor is safe
```

---

## Complete Workflow Example

Here's the full cycle applied to a `ShoppingCart` module:

```typescript
// 1. RED — Human writes all tests upfront
describe('ShoppingCart', () => {
  let cart: ShoppingCart;

  beforeEach(() => {
    cart = new ShoppingCart();
  });

  it('should start empty', () => {
    expect(cart.items).toEqual([]);
    expect(cart.total).toBe(0);
  });

  it('should add items', () => {
    cart.addItem({ id: '1', name: 'Widget', price: 9.99, quantity: 2 });
    expect(cart.items).toHaveLength(1);
    expect(cart.total).toBe(19.98);
  });

  it('should remove items', () => {
    cart.addItem({ id: '1', name: 'Widget', price: 9.99, quantity: 1 });
    cart.removeItem('1');
    expect(cart.items).toEqual([]);
    expect(cart.total).toBe(0);
  });

  it('should apply percentage discounts', () => {
    cart.addItem({ id: '1', name: 'Widget', price: 100, quantity: 1 });
    cart.applyDiscount({ type: 'percentage', value: 10 });
    expect(cart.total).toBe(90);
  });

  it('should not allow negative totals', () => {
    cart.addItem({ id: '1', name: 'Widget', price: 10, quantity: 1 });
    cart.applyDiscount({ type: 'fixed', value: 50 });
    expect(cart.total).toBe(0);
  });
});
```

```
// 2. GREEN — Give tests to AI: "Implement ShoppingCart to pass these tests"
// 3. REFACTOR — Review AI output, ask for improvements while tests guard
```

---

## Benefits of TDD with AI

| Benefit | Why It Matters |
|---------|---------------|
| **Tests verify AI correctness** | AI can hallucinate plausible but wrong code — tests catch this |
| **AI fills implementation fast** | The slowest part of TDD (writing code) becomes nearly instant |
| **Human maintains design control** | You define the API surface through tests |
| **Refactoring is safe** | Tests protect against regressions during AI-assisted refactoring |
| **Context is compressed** | Tests are a compact specification — more efficient than prose descriptions |

---

## Tips for Effective AI TDD

- ✅ **Write tests that are specific and assertion-rich** — vague tests produce vague implementations
- ✅ **Run tests after every AI generation** — never trust, always verify
- ✅ **Include type definitions** in your test file so the AI knows the expected interface
- ✅ **Start with simple cases**, add complex ones incrementally
- ❌ **Don't skip the RED step** — if you don't verify the test fails first, you can't trust the GREEN step
- ❌ **Don't let AI write both tests and implementation** — that defeats the purpose of TDD
