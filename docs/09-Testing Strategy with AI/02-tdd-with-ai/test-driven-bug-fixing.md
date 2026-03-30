# Test-Driven Bug Fixing

> Write the failing test first, then let AI fix the bug — the most reliable AI bug-fixing workflow.

---

## Why "Fix This Bug" Doesn't Work

When you tell an AI "fix this bug," you're relying on the AI to:

1. Understand the expected behavior (ambiguous)
2. Understand the actual behavior (requires running the code)
3. Identify the root cause (requires deep context)
4. Produce a correct fix (error-prone without verification)

When you give the AI a **failing test + the code**, you're giving it:

1. ✅ An unambiguous specification of correct behavior (the test)
2. ✅ Clear evidence of the bug (the test fails)
3. ✅ A verification mechanism (run the test after the fix)
4. ✅ Regression protection (the test stays in the suite)

```
❌ "Fix this bug"
   → AI guesses what "correct" means
   → No way to verify the fix is right
   → Might introduce new bugs

✅ "Make this failing test pass"
   → AI knows exactly what "correct" means
   → Test verifies the fix
   → Test prevents regression
```

---

## The Four-Step Workflow

This applies the Red-Green-Refactor cycle from [TDD Workflow](tdd-workflow.md) to bug fixing: reproduce → fix → verify.

### Step 1: Reproduce the Bug as a Failing Test

Before touching the code, write a test that captures the exact broken behavior.

```typescript
// Bug report: "Users with special characters in names get corrupted display"
// Step 1: Write a test that reproduces the bug

describe('formatUserDisplayName', () => {
  // Existing tests (all pass)
  it('should format a simple name', () => {
    expect(formatUserDisplayName('John', 'Doe')).toBe('John Doe');
  });

  // NEW: Test that reproduces the bug
  it('should handle names with apostrophes', () => {
    expect(formatUserDisplayName("O'Brien", 'Connor')).toBe("O'Brien Connor");
  });

  it('should handle names with hyphens', () => {
    expect(formatUserDisplayName('Mary', 'Smith-Jones')).toBe('Mary Smith-Jones');
  });

  it('should handle names with accented characters', () => {
    expect(formatUserDisplayName('José', 'García')).toBe('José García');
  });
});
```

```bash
npx jest format-user.test.ts
# PASS: should format a simple name ✓
# FAIL: should handle names with apostrophes ✗
#   Expected: "O'Brien Connor"
#   Received: "OBrien Connor"
# FAIL: should handle names with accented characters ✗
#   Expected: "José García"
#   Received: "Jos Garca"
```

**Now you have proof of the bug and a clear target for the fix.**

### Step 2: Give the AI the Failing Test + Source Code

```
The following test is failing. Fix the `formatUserDisplayName` function
so all tests pass, including the existing ones. Do not change the tests.

## Failing test output:
[paste test results]

## Current implementation:
[paste source file]

## Test file:
[paste test file]
```

### Step 3: AI Fixes the Code

**Before (buggy):**

```typescript
// format-user.ts — BUGGY VERSION
export function formatUserDisplayName(first: string, last: string): string {
  // Bug: sanitize was stripping non-ASCII and special characters
  const sanitized = (first + ' ' + last).replace(/[^a-zA-Z0-9 ]/g, '');
  return sanitized.trim();
}
```

**After (AI-fixed):**

```typescript
// format-user.ts — FIXED VERSION
export function formatUserDisplayName(first: string, last: string): string {
  const sanitized = (first + ' ' + last)
    .replace(/[<>{}]/g, '')  // Only remove dangerous characters
    .trim();
  return sanitized;
}
```

### Step 4: Verify — Run the Full Test Suite

```bash
npx jest format-user.test.ts
# PASS: should format a simple name ✓
# PASS: should handle names with apostrophes ✓
# PASS: should handle names with hyphens ✓
# PASS: should handle names with accented characters ✓

# Run full suite to check for regressions
npx jest
# All 142 tests passed ✓
```

---

## Complete Before/After Example

### Bug: Discount calculation breaks for quantities above 100

**Step 1: Reproduce**

```typescript
// discount-calculator.test.ts
describe('calculateDiscount', () => {
  // Existing passing tests
  it('should apply 10% discount for 10+ items', () => {
    expect(calculateDiscount(10, 50)).toBe(450); // 10 × $50 × 0.90
  });

  it('should apply 20% discount for 50+ items', () => {
    expect(calculateDiscount(50, 50)).toBe(2000); // 50 × $50 × 0.80
  });

  // BUG: This returns NaN instead of correct value
  it('should apply 30% discount for 100+ items', () => {
    expect(calculateDiscount(100, 50)).toBe(3500); // 100 × $50 × 0.70
  });

  it('should apply 30% discount for 200 items', () => {
    expect(calculateDiscount(200, 25)).toBe(3500); // 200 × $25 × 0.70
  });

  // Edge case from bug report
  it('should handle exactly 100 items', () => {
    expect(calculateDiscount(100, 10)).toBe(700); // 100 × $10 × 0.70
  });
});
```

```bash
npx jest discount-calculator.test.ts
# PASS: 10% discount ✓
# PASS: 20% discount ✓
# FAIL: 30% for 100+ items — Expected 3500, Received NaN
# FAIL: 30% for 200 items — Expected 3500, Received NaN
# FAIL: exactly 100 items — Expected 700, Received NaN
```

**Step 2: Give to AI with context**

```
These tests are failing with NaN for quantities >= 100.
Fix the function. Don't change the tests.

[paste test file + source file]
```

**Step 3: AI identifies and fixes the bug**

```typescript
// BEFORE — buggy
export function calculateDiscount(quantity: number, unitPrice: number): number {
  const discountTiers = [
    { min: 50, rate: 0.20 },
    { min: 10, rate: 0.10 },
  ];

  const tier = discountTiers.find((t) => quantity >= t.min);
  // BUG: No tier matched for 100+ → tier is undefined → NaN
  return quantity * unitPrice * (1 - tier.rate);
}
```

```typescript
// AFTER — fixed
export function calculateDiscount(quantity: number, unitPrice: number): number {
  const discountTiers = [
    { min: 100, rate: 0.30 },
    { min: 50, rate: 0.20 },
    { min: 10, rate: 0.10 },
  ];

  const tier = discountTiers.find((t) => quantity >= t.min);
  const rate = tier ? tier.rate : 0;
  return quantity * unitPrice * (1 - rate);
}
```

**Step 4: Verify**

```bash
npx jest discount-calculator.test.ts
# PASS: 5/5 tests passing ✓
```

---

## When to Use This Pattern

| Scenario | Use Test-Driven Bug Fix? |
|----------|------------------------|
| Bug with clear reproduction steps | ✅ Yes — ideal case |
| Bug from a user report | ✅ Yes — translate report into failing test |
| Intermittent / flaky bug | ⚠️ Maybe — if you can make it deterministic in a test |
| Performance regression | ⚠️ Maybe — if you can express as a timing assertion |
| Visual / UI bug | ❌ Usually not — use screenshot comparison instead |
| "Something feels wrong" | ❌ No — first define what "correct" means, then write the test |

---

## Tips for Writing Bug-Reproducing Tests

- **Be specific about inputs and outputs** — the test should fail for the exact reason the bug manifests
- **Include the boundary condition** — if the bug triggers at quantity 100, test at 99, 100, and 101
- **Name the test after the bug** — `it('should not return NaN for bulk quantities (BUG-1234)')` makes the test traceable
- **Keep existing tests** — never remove or modify passing tests; only add new ones
- **Test the fix, not the implementation** — assert the correct output, not that a specific line changed
