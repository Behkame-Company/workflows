# AI Test Generation Patterns

> Comprehensive prompt patterns for generating tests — from function signatures to user stories, with templates and examples for each.

---

## Choosing the Right Pattern

| Pattern | Input | Best For | Quality |
|---------|-------|----------|---------|
| [From Function Signature](#pattern-1-from-function-signature) | Function name + types | Quick unit tests for utility functions | ⭐⭐⭐ |
| [From Existing Code](#pattern-2-from-existing-code) | Complete implementation | Adding tests to untested code | ⭐⭐⭐⭐ |
| [From Spec / Interface](#pattern-3-from-spec--interface) | TypeScript interfaces, OpenAPI | Contract-first development | ⭐⭐⭐⭐⭐ |
| [From Bug Report](#pattern-4-from-bug-report) | Issue description + code | Regression tests for fixes | ⭐⭐⭐⭐ |
| [From User Story](#pattern-5-from-user-story) | Acceptance criteria | E2E and integration tests | ⭐⭐⭐ |

---

## Pattern 1: From Function Signature

**When to use:** You have a function signature (or just a name and expected types) but no implementation yet.

### Prompt Template

```
Generate comprehensive tests for a function with this signature:

Function: {functionName}
Parameters: {parameterList}
Returns: {returnType}
Purpose: {briefDescription}

Include tests for:
- Happy path with typical inputs
- Edge cases for each parameter
- Error/validation cases
- Return type verification

Use {testFramework} with {language}.
```

### Example

**Prompt:**
```
Generate comprehensive tests for a function with this signature:

Function: slugify
Parameters: input: string, options?: { separator?: string; lowercase?: boolean }
Returns: string
Purpose: Converts a string to a URL-friendly slug

Include tests for:
- Happy path with typical inputs
- Edge cases for each parameter
- Error/validation cases
- Return type verification

Use Jest with TypeScript.
```

**Generated Tests:**

```typescript
import { slugify } from './slugify';

describe('slugify', () => {
  // Happy path
  it('should convert a simple string to a slug', () => {
    expect(slugify('Hello World')).toBe('hello-world');
  });

  it('should handle multiple spaces', () => {
    expect(slugify('Hello   World')).toBe('hello-world');
  });

  it('should remove special characters', () => {
    expect(slugify('Hello, World!')).toBe('hello-world');
  });

  // Options
  it('should use custom separator', () => {
    expect(slugify('Hello World', { separator: '_' })).toBe('hello_world');
  });

  it('should preserve case when lowercase is false', () => {
    expect(slugify('Hello World', { lowercase: false })).toBe('Hello-World');
  });

  // Edge cases
  it('should return empty string for empty input', () => {
    expect(slugify('')).toBe('');
  });

  it('should handle string with only special characters', () => {
    expect(slugify('!@#$%')).toBe('');
  });

  it('should handle leading and trailing spaces', () => {
    expect(slugify('  Hello World  ')).toBe('hello-world');
  });

  it('should handle unicode characters', () => {
    expect(slugify('Héllo Wörld')).toBe('hello-world');
  });

  it('should handle numbers', () => {
    expect(slugify('Product 42 Pro')).toBe('product-42-pro');
  });

  it('should not produce consecutive separators', () => {
    expect(slugify('Hello -- World')).toBe('hello-world');
  });
});
```

---

## Pattern 2: From Existing Code

**When to use:** You have working code that lacks tests. This is the most common scenario when adding tests to legacy code.

### Prompt Template

```
Generate tests for the following {language} function.
Analyze the implementation to identify all code paths, branches,
and edge cases. Generate a test for each path.

Do NOT modify the function. Only generate tests.

```{language}
{pasteImplementation}
```

Use {testFramework}. Cover:
- Every conditional branch
- Every error condition
- Boundary values for numeric comparisons
- Null/undefined handling
```

### Example

**Input code:**
```typescript
export function calculateShipping(
  weight: number,
  destination: 'domestic' | 'international',
  expedited: boolean
): number {
  if (weight <= 0) throw new Error('Weight must be positive');
  if (weight > 50) throw new Error('Weight exceeds maximum');

  let base: number;
  if (destination === 'domestic') {
    base = weight <= 5 ? 5.99 : weight <= 20 ? 12.99 : 24.99;
  } else {
    base = weight <= 5 ? 14.99 : weight <= 20 ? 29.99 : 59.99;
  }

  return expedited ? base * 1.5 : base;
}
```

**Generated Tests:**

```typescript
describe('calculateShipping', () => {
  // Error conditions
  it('should throw for zero weight', () => {
    expect(() => calculateShipping(0, 'domestic', false)).toThrow('Weight must be positive');
  });

  it('should throw for negative weight', () => {
    expect(() => calculateShipping(-1, 'domestic', false)).toThrow('Weight must be positive');
  });

  it('should throw for weight over 50', () => {
    expect(() => calculateShipping(51, 'domestic', false)).toThrow('Weight exceeds maximum');
  });

  // Domestic tiers
  it('should charge $5.99 for domestic <= 5kg', () => {
    expect(calculateShipping(3, 'domestic', false)).toBe(5.99);
  });

  it('should charge $12.99 for domestic 5-20kg', () => {
    expect(calculateShipping(10, 'domestic', false)).toBe(12.99);
  });

  it('should charge $24.99 for domestic 20-50kg', () => {
    expect(calculateShipping(30, 'domestic', false)).toBe(24.99);
  });

  // International tiers
  it('should charge $14.99 for international <= 5kg', () => {
    expect(calculateShipping(3, 'international', false)).toBe(14.99);
  });

  it('should charge $29.99 for international 5-20kg', () => {
    expect(calculateShipping(10, 'international', false)).toBe(29.99);
  });

  it('should charge $59.99 for international 20-50kg', () => {
    expect(calculateShipping(30, 'international', false)).toBe(59.99);
  });

  // Expedited multiplier
  it('should apply 1.5x multiplier for expedited domestic', () => {
    expect(calculateShipping(3, 'domestic', true)).toBeCloseTo(8.985);
  });

  it('should apply 1.5x multiplier for expedited international', () => {
    expect(calculateShipping(3, 'international', true)).toBeCloseTo(22.485);
  });

  // Boundary values
  it('should use lower tier at exactly 5kg', () => {
    expect(calculateShipping(5, 'domestic', false)).toBe(5.99);
  });

  it('should use middle tier at 5.01kg', () => {
    expect(calculateShipping(5.01, 'domestic', false)).toBe(12.99);
  });

  it('should use middle tier at exactly 20kg', () => {
    expect(calculateShipping(20, 'domestic', false)).toBe(12.99);
  });

  it('should accept exactly 50kg', () => {
    expect(calculateShipping(50, 'domestic', false)).toBe(24.99);
  });
});
```

---

## Pattern 3: From Spec / Interface

**When to use:** You have a formal specification (TypeScript interface, OpenAPI schema, JSON Schema) and want to generate tests before implementation.

### Prompt Template

```
Given this {specType}, generate exhaustive tests for an implementation.

```{language}
{pasteSpec}
```

For each method/endpoint:
- Test the happy path
- Test every parameter type and constraint
- Test error responses
- Test optional vs required fields
- Test enum values exhaustively

Use {testFramework}.
```

See [Spec-Driven Testing](../02-tdd-with-ai/spec-driven-testing.md) for full examples.

---

## Pattern 4: From Bug Report

**When to use:** You have a bug report and want to create a regression test before fixing it.

### Prompt Template

```
Generate a regression test for this bug:

Bug: {bugDescription}
Steps to reproduce: {steps}
Expected behavior: {expected}
Actual behavior: {actual}

Affected code:
```{language}
{pasteCode}
```

Generate:
1. A test that reproduces the bug (currently fails)
2. Additional edge case tests for related scenarios
Use {testFramework}.
```

### Example

**Bug report:**
```
Bug: formatCurrency shows "-$0.00" for zero values
Steps: Call formatCurrency(0) or formatCurrency(-0)
Expected: "$0.00"
Actual: "-$0.00"
```

**Generated Tests:**

```typescript
describe('formatCurrency - zero handling (BUG-892)', () => {
  it('should format positive zero as "$0.00"', () => {
    expect(formatCurrency(0)).toBe('$0.00');
  });

  it('should format negative zero as "$0.00"', () => {
    expect(formatCurrency(-0)).toBe('$0.00');
  });

  it('should format very small negative as "$0.00" when rounded', () => {
    expect(formatCurrency(-0.001)).toBe('$0.00');
  });

  it('should still format actual negatives correctly', () => {
    expect(formatCurrency(-5.99)).toBe('-$5.99');
  });
});
```

---

## Pattern 5: From User Story

**When to use:** You have acceptance criteria from a user story and want to generate acceptance tests.

### Prompt Template

```
Generate test cases from this user story:

As a {role}
I want to {action}
So that {benefit}

Acceptance Criteria:
{listCriteria}

Generate one test per acceptance criterion plus edge cases.
Use {testFramework}. Tests should be readable as documentation.
```

### Example

**User Story:**
```
As a shopper
I want to apply discount codes at checkout
So that I can get promotional pricing

Acceptance Criteria:
- Valid codes reduce the total by the discount amount
- Percentage codes apply to subtotal before tax
- Fixed-amount codes cannot reduce total below zero
- Expired codes show an error message
- Each code can only be used once per order
```

**Generated Tests:**

```typescript
describe('Checkout: Discount Codes', () => {
  // "Valid codes reduce the total by the discount amount"
  it('should reduce total when a valid percentage code is applied', () => {
    const cart = createCart([{ price: 100, qty: 1 }]);
    cart.applyCode('SAVE10'); // 10% off
    expect(cart.total).toBe(90);
  });

  it('should reduce total when a valid fixed-amount code is applied', () => {
    const cart = createCart([{ price: 100, qty: 1 }]);
    cart.applyCode('OFF15'); // $15 off
    expect(cart.total).toBe(85);
  });

  // "Percentage codes apply to subtotal before tax"
  it('should calculate percentage discount on subtotal, not total with tax', () => {
    const cart = createCart([{ price: 100, qty: 1 }]);
    cart.setTaxRate(0.10); // 10% tax
    cart.applyCode('SAVE10'); // 10% off subtotal
    expect(cart.subtotal).toBe(90);
    expect(cart.tax).toBe(9); // tax on discounted subtotal
    expect(cart.total).toBe(99);
  });

  // "Fixed-amount codes cannot reduce total below zero"
  it('should not reduce total below zero with large fixed discount', () => {
    const cart = createCart([{ price: 10, qty: 1 }]);
    cart.applyCode('OFF50'); // $50 off, but cart is only $10
    expect(cart.total).toBe(0);
  });

  // "Expired codes show an error message"
  it('should reject expired discount codes', () => {
    const cart = createCart([{ price: 100, qty: 1 }]);
    const result = cart.applyCode('EXPIRED2023');
    expect(result.success).toBe(false);
    expect(result.error).toContain('expired');
  });

  // "Each code can only be used once per order"
  it('should reject duplicate code application', () => {
    const cart = createCart([{ price: 100, qty: 1 }]);
    cart.applyCode('SAVE10');
    const result = cart.applyCode('SAVE10');
    expect(result.success).toBe(false);
    expect(result.error).toContain('already applied');
  });
});
```

---

## Combining Patterns

For complex features, combine patterns for thorough coverage:

```
1. Start with Pattern 3 (Spec/Interface) for contract tests
2. Add Pattern 5 (User Story) for acceptance tests
3. Fill gaps with Pattern 1 (Signature) for utility functions
4. Use Pattern 4 (Bug Report) as bugs surface
5. Retrofit with Pattern 2 (Existing Code) for untested code
```
