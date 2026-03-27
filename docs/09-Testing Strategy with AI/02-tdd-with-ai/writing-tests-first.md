# Writing Tests First for AI

> The more specific your tests, the less room AI has for error — tests are the strongest constraints you can give an AI agent.

---

## Tests as Constraints

When you ask an AI to "implement an email validator," you get whatever the model thinks is reasonable. When you give it **five specific tests**, you get code that satisfies exactly those five behaviors.

```
Vague prompt:       "Write an email validator"
                    → AI decides what "valid" means
                    → May miss edge cases you care about
                    → May add behavior you don't want

Tests-first prompt: "Make these 5 tests pass"
                    → AI implements exactly what you specified
                    → Edge cases are covered (you wrote them)
                    → No unwanted behavior (tests constrain output)
```

**Tests are executable specifications.** Unlike prose requirements that AI can misinterpret, tests have a binary pass/fail result.

---

## Principles of Test-First for AI

### 1. Write Assertion-Rich Tests

Every assertion is a constraint. More assertions = tighter specification = better AI output.

```typescript
// ❌ Weak constraint — AI has too much freedom
it('should parse a date', () => {
  const result = parseDate('2024-01-15');
  expect(result).toBeDefined();
});

// ✅ Strong constraint — AI knows exactly what to produce
it('should parse ISO date string into components', () => {
  const result = parseDate('2024-01-15');
  expect(result.year).toBe(2024);
  expect(result.month).toBe(1);
  expect(result.day).toBe(15);
  expect(result.isValid).toBe(true);
  expect(result.format('MM/DD/YYYY')).toBe('01/15/2024');
});
```

### 2. Include Edge Cases Before Generation

Don't wait for the AI to miss edge cases — write them into your test suite upfront.

```typescript
// Include edge cases alongside happy path BEFORE asking AI to implement
describe('divideNumbers', () => {
  // Happy path
  it('should divide two positive numbers', () => {
    expect(divideNumbers(10, 2)).toBe(5);
  });

  // Edge cases — written first, not afterthoughts
  it('should handle division by zero', () => {
    expect(() => divideNumbers(10, 0)).toThrow('Division by zero');
  });

  it('should handle negative numbers', () => {
    expect(divideNumbers(-10, 2)).toBe(-5);
  });

  it('should handle floating point results', () => {
    expect(divideNumbers(1, 3)).toBeCloseTo(0.3333, 4);
  });

  it('should handle very large numbers', () => {
    expect(divideNumbers(Number.MAX_SAFE_INTEGER, 1)).toBe(Number.MAX_SAFE_INTEGER);
  });
});
```

### 3. Order Tests from Simple to Complex

AI agents that implement incrementally (test by test) produce better results when tests progress in difficulty:

```typescript
describe('MarkdownParser', () => {
  // Level 1: Basic structure
  it('should return empty string for empty input', () => {
    expect(parse('')).toBe('');
  });

  // Level 2: Single feature
  it('should parse bold text', () => {
    expect(parse('**hello**')).toBe('<strong>hello</strong>');
  });

  // Level 3: Combined features
  it('should parse bold inside a paragraph', () => {
    expect(parse('This is **bold** text')).toBe('<p>This is <strong>bold</strong> text</p>');
  });

  // Level 4: Edge cases
  it('should handle unclosed bold markers', () => {
    expect(parse('**unclosed')).toBe('<p>**unclosed</p>');
  });

  // Level 5: Complex combinations
  it('should parse nested formatting', () => {
    expect(parse('**bold and *italic***')).toBe(
      '<strong>bold and <em>italic</em></strong>'
    );
  });
});
```

### 4. Define Return Types Through Tests

Use test assertions to implicitly define the shape of return values:

```typescript
// These tests tell the AI exactly what the return type looks like
describe('analyzeText', () => {
  it('should return word count', () => {
    const result = analyzeText('hello world');
    expect(result.wordCount).toBe(2);
  });

  it('should return character count without spaces', () => {
    const result = analyzeText('hello world');
    expect(result.charCount).toBe(10);
  });

  it('should return sentence count', () => {
    const result = analyzeText('Hello. World.');
    expect(result.sentenceCount).toBe(2);
  });

  it('should return average word length', () => {
    const result = analyzeText('hi there');
    expect(result.avgWordLength).toBe(3.5);
  });

  it('should return the full analysis object', () => {
    const result = analyzeText('Hello world.');
    expect(result).toEqual({
      wordCount: 2,
      charCount: 10,
      sentenceCount: 1,
      avgWordLength: 5,
    });
  });
});
```

---

## Complete Example: Email Validator

Here's the full workflow — write 5 tests, then prompt AI to implement.

### Step 1: Write the Tests

```typescript
// email-validator.test.ts
import { validateEmail } from './email-validator';

describe('validateEmail', () => {
  // Test 1: Valid standard email
  it('should accept a valid email address', () => {
    expect(validateEmail('user@example.com')).toEqual({
      valid: true,
      reason: null,
    });
  });

  // Test 2: Missing @ symbol
  it('should reject email without @ symbol', () => {
    expect(validateEmail('userexample.com')).toEqual({
      valid: false,
      reason: 'Missing @ symbol',
    });
  });

  // Test 3: Missing domain
  it('should reject email without domain', () => {
    expect(validateEmail('user@')).toEqual({
      valid: false,
      reason: 'Missing domain',
    });
  });

  // Test 4: Missing local part
  it('should reject email without local part', () => {
    expect(validateEmail('@example.com')).toEqual({
      valid: false,
      reason: 'Missing local part',
    });
  });

  // Test 5: Invalid domain format
  it('should reject email with invalid domain', () => {
    expect(validateEmail('user@.com')).toEqual({
      valid: false,
      reason: 'Invalid domain format',
    });
  });
});
```

### Step 2: Verify Tests Fail

```bash
npx jest email-validator.test.ts
# FAIL: Cannot find module './email-validator'
# 5 tests failed ✓ (this is expected)
```

### Step 3: Prompt AI

```
I have the following test file for an email validator.
Implement the `validateEmail` function in `email-validator.ts`
to make all tests pass. Export the function. Do not add any
behavior beyond what the tests require.

[paste email-validator.test.ts]
```

### Step 4: AI Generates Implementation

```typescript
// email-validator.ts — AI-generated
interface ValidationResult {
  valid: boolean;
  reason: string | null;
}

export function validateEmail(email: string): ValidationResult {
  if (!email.includes('@')) {
    return { valid: false, reason: 'Missing @ symbol' };
  }

  const [local, ...domainParts] = email.split('@');
  const domain = domainParts.join('@');

  if (!local) {
    return { valid: false, reason: 'Missing local part' };
  }

  if (!domain) {
    return { valid: false, reason: 'Missing domain' };
  }

  if (domain.startsWith('.') || !domain.includes('.')) {
    return { valid: false, reason: 'Invalid domain format' };
  }

  return { valid: true, reason: null };
}
```

### Step 5: Run and Verify

```bash
npx jest email-validator.test.ts
# PASS: 5/5 tests passing ✓
```

---

## Test-First Checklist

Before handing tests to an AI agent, verify:

| Check | Why |
|-------|-----|
| ✅ Tests fail without implementation | Ensures tests actually test something |
| ✅ Each test has specific assertions | Vague assertions produce vague code |
| ✅ Edge cases are included | AI won't handle what you don't test |
| ✅ Return types are implied by assertions | AI infers the data shape from tests |
| ✅ Tests are ordered simple → complex | Incremental implementation is more reliable |
| ✅ Tests are independent (no shared state) | Each test constrains one behavior |

---

## Anti-Patterns to Avoid

```typescript
// ❌ Testing implementation details — too brittle
it('should use a regex for validation', () => {
  expect(validateEmail.toString()).toContain('RegExp');
});

// ❌ Testing too many things in one assertion — unclear failures
it('should validate emails', () => {
  expect(validateEmail('a@b.com').valid).toBe(true);
  expect(validateEmail('bad').valid).toBe(false);
  expect(validateEmail('@x.com').valid).toBe(false);
  // If this fails, which case broke?
});

// ✅ One behavior per test — clear signal on failure
it('should accept valid email with subdomain', () => {
  expect(validateEmail('user@mail.example.com')).toEqual({
    valid: true,
    reason: null,
  });
});
```

---

## Next Steps

- [TDD Workflow with AI Agents](tdd-workflow.md) — the full Red-Green-Refactor cycle with AI
- [Test-Driven Bug Fixing](test-driven-bug-fixing.md) — applying test-first to debugging
- [AI Test Generation Patterns](../03-test-generation/generation-patterns.md) — when AI writes the tests instead
- [Test as Specification](../01-fundamentals/test-as-specification.md) — the philosophy behind tests as constraints

---

*Part of the [Testing Strategy with AI](../README.md) documentation series.*
