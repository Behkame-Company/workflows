# Mutation Testing

> Testing your tests by mutating code — the most rigorous way to verify that your test assertions are strong enough to catch real bugs.

## What Is Mutation Testing?

Mutation testing answers: **"If I introduced a bug, would my tests catch it?"**

The process:

1. **Mutate** — Automatically change your source code in small ways (called "mutants")
2. **Run tests** — Execute your test suite against each mutant
3. **Check results** — If tests fail, the mutant is "killed" (good). If tests pass, the mutant "survived" (bad — your tests missed a real bug)

### Example

```typescript
// Original code
function isEligible(age: number, score: number): boolean {
  return age >= 18 && score > 70;
}
```

Mutation testing generates mutants like:

| Mutant | Change | Tests Should... |
|--------|--------|----------------|
| `age > 18` | Changed `>=` to `>` | Fail (catch the off-by-one) |
| `age >= 18 \|\| score > 70` | Changed `&&` to `\|\|` | Fail (catch the logic change) |
| `score >= 70` | Changed `>` to `>=` | Fail (catch boundary change) |
| `return true` | Replaced entire expression | Fail (catch the removal) |

If any mutant **survives** (tests still pass), your tests have a gap.

## Mutation Score

```
Mutation Score = (Killed Mutants / Total Mutants) × 100%
```

| Score | Quality |
|-------|---------|
| 90%+ | Excellent — tests catch almost every possible bug |
| 70-89% | Good — strong test suite with some gaps |
| 50-69% | Fair — significant gaps in assertion coverage |
| < 50% | Poor — tests provide false confidence |

## Tools

| Language | Tool | Install |
|----------|------|---------|
| JavaScript/TypeScript | [Stryker](https://stryker-mutator.io/) | `npm install --save-dev @stryker-mutator/core` |
| Python | [mutmut](https://mutmut.readthedocs.io/) | `pip install mutmut` |
| Java | [PIT](https://pitest.org/) | Maven/Gradle plugin |
| C# | [Stryker.NET](https://stryker-mutator.io/docs/stryker-net/introduction/) | `dotnet tool install dotnet-stryker` |
| Rust | [cargo-mutants](https://github.com/sourcefrog/cargo-mutants) | `cargo install cargo-mutants` |

## Stryker Setup (JavaScript/TypeScript)

### Installation

```bash
npm install --save-dev @stryker-mutator/core @stryker-mutator/jest-runner @stryker-mutator/typescript-checker
npx stryker init
```

### Configuration

```javascript
// stryker.config.mjs
/** @type {import('@stryker-mutator/api/core').PartialStrykerOptions} */
export default {
  mutate: ['src/**/*.ts', '!src/**/*.test.ts', '!src/**/*.spec.ts'],
  testRunner: 'jest',
  checkers: ['typescript'],
  reporters: ['clear-text', 'html'],
  coverageAnalysis: 'perTest',
  thresholds: {
    high: 80,
    low: 60,
    break: 50, // Fail CI if score drops below 50%
  },
};
```

### Running

```bash
# Full mutation run
npx stryker run

# Mutate only specific files (faster for CI)
npx stryker run --mutate "src/services/order-service.ts"
```

### Reading the Report

```
Mutation testing  [====================] 100% (elapsed: 2m, remaining: ~0s)
  61 Mutant(s) tested

Kill:    47 ✅
Survived: 8 ❌
Timeout:  3
No coverage: 3

Mutation score: 83.93%

Survived mutants:
  src/order-service.ts:45:12 — StringLiteral: replaced "rejected" with ""
  src/order-service.ts:52:5  — ConditionalExpression: removed conditional
  src/order-service.ts:67:20 — ArithmeticOperator: replaced * with /
  ...
```

Each survived mutant tells you **exactly where your tests are weak**.

## AI-Generated Tests and Mutation Testing

AI-generated tests often have **low mutation scores** because they tend to:

- Use weak assertions (`toBeDefined()`, `toBeTruthy()`)
- Test that code runs without checking outputs
- Miss boundary conditions
- Skip negative test cases

### The AI + Mutation Testing Pipeline

```
Step 1: AI generates tests
  ↓
Step 2: Run mutation testing → Find survivors
  ↓
Step 3: Show survivors to AI → "Strengthen tests to kill these mutants"
  ↓
Step 4: Run mutation testing again → Verify improvement
  ↓
Repeat until mutation score meets threshold
```

### Prompt for Strengthening Tests

```
These mutants survived my test suite for `calculateShipping()`:

1. Line 12: Changed `weight > 50` to `weight >= 50`
   — Tests don't check the boundary at weight=50

2. Line 18: Changed `+ handlingFee` to `- handlingFee`
   — Tests don't verify the handling fee is added correctly

3. Line 25: Replaced `"express"` with `""`
   — Tests don't verify the shipping method string

For each surviving mutant, write a test that would catch (kill) it.
Use exact boundary values and specific assertion values.
```

### AI Response: Targeted Tests

```typescript
describe('calculateShipping — mutation-killing tests', () => {
  // Kills mutant 1: boundary at weight=50
  it('should apply heavy package surcharge at exactly 50kg', () => {
    const resultAt50 = calculateShipping({ weight: 50, method: 'standard' });
    const resultAt49 = calculateShipping({ weight: 49, method: 'standard' });

    // 50 should NOT trigger surcharge (> 50, not >= 50)
    expect(resultAt50).toBe(resultAt49);

    // 51 should trigger surcharge
    const resultAt51 = calculateShipping({ weight: 51, method: 'standard' });
    expect(resultAt51).toBeGreaterThan(resultAt50);
  });

  // Kills mutant 2: handling fee arithmetic
  it('should add handling fee to base shipping cost', () => {
    const result = calculateShipping({
      weight: 10,
      method: 'standard',
      handlingFee: 5.00,
    });

    // Base cost for 10kg standard = $12.00
    // With $5 handling fee = $17.00
    expect(result).toBe(17.00);
  });

  // Kills mutant 3: shipping method string
  it('should set shipping method to "express" for express delivery', () => {
    const result = calculateShippingDetails({
      weight: 10,
      method: 'express',
    });

    expect(result.method).toBe('express');
    expect(result.method).not.toBe('');
  });
});
```

## mutmut Example (Python)

```bash
# Run mutation testing
mutmut run --paths-to-mutate src/

# See results
mutmut results

# Show a specific surviving mutant
mutmut show 42

# Output:
# --- src/pricing.py
# +++ src/pricing.py (mutant 42)
# @@ -15,7 +15,7 @@
#  def apply_discount(price, discount_pct):
# -    return price * (1 - discount_pct / 100)
# +    return price * (1 + discount_pct / 100)
```

Then write a test that catches mutant 42:

```python
def test_discount_reduces_price():
    """Kills mutant: changing subtraction to addition in discount calc."""
    original_price = 100.00
    result = apply_discount(original_price, 20)

    assert result == 80.00  # 20% off $100
    assert result < original_price  # discount must reduce price
```

## Mutation Testing in CI

Running full mutation testing on every PR is often too slow. Use these strategies:

### Strategy 1: Mutate Only Changed Files

```yaml
# GitHub Actions — run mutations on changed files only
- name: Mutation test changed files
  run: |
    CHANGED=$(git diff --name-only origin/main...HEAD -- 'src/**/*.ts' | tr '\n' ',')
    if [ -n "$CHANGED" ]; then
      npx stryker run --mutate "$CHANGED"
    fi
```

### Strategy 2: Scheduled Full Runs

```yaml
# Run full mutation testing weekly
on:
  schedule:
    - cron: '0 2 * * 1' # Monday 2 AM

jobs:
  mutation-testing:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npx stryker run
      - name: Upload mutation report
        uses: actions/upload-artifact@v4
        with:
          name: mutation-report
          path: reports/mutation/
```

### Strategy 3: Gate Critical Paths Only

```javascript
// stryker.config.mjs — only mutate critical business logic
export default {
  mutate: [
    'src/services/payment-service.ts',
    'src/services/order-service.ts',
    'src/core/pricing/**/*.ts',
  ],
  thresholds: {
    break: 70, // Block PR if mutation score drops below 70%
  },
};
```
