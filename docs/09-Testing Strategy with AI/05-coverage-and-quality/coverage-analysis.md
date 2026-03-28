# Coverage Analysis

> Measuring and improving test coverage with AI — understanding what coverage metrics actually tell you, where the gaps are, and how AI can fill them efficiently.

## What Coverage Measures

Coverage tells you which parts of your code were **executed** during tests. It does not tell you whether the tests **verified** that the code worked correctly.

### Coverage Types

| Type | What It Measures | Example |
|------|-----------------|---------|
| **Line coverage** | Which lines were executed | Was line 42 reached? |
| **Branch coverage** | Which conditional branches were taken | Was the `else` path of the `if` on line 15 tested? |
| **Function coverage** | Which functions were called | Was `validateEmail()` invoked at all? |
| **Path coverage** | Which combinations of branches were taken | Was the path where both `if` on line 10 and `else` on line 20 were hit? |
| **Statement coverage** | Which statements were executed | Similar to line coverage but handles multiple statements per line |

### The Coverage Hierarchy

```
Path coverage (most thorough, hardest to achieve)
  ⊃ Branch coverage (catches missed conditionals)
    ⊃ Line coverage (most commonly measured)
      ⊃ Function coverage (bare minimum)
```

**Branch coverage is the sweet spot** for most projects — it catches missed conditional paths without the exponential explosion of full path coverage.

## Coverage Targets

| Target | When to Use |
|--------|-------------|
| **80% line + branch** | Good default for most projects |
| **90%+ line** | Critical libraries, shared packages, core business logic |
| **100% line** | Usually wasteful — diminishing returns on the last 10-20% |
| **60-70%** | Acceptable for legacy code being gradually improved |

### Why 100% Coverage Is Usually Wasteful

```typescript
// Getting 100% coverage means testing code like this:
catch (error) {
  // This only fires if the OS runs out of file descriptors
  logger.error('Catastrophic failure', error);
  process.exit(1);
}
```

The effort to test that catch block (mocking OS-level failures) vastly outweighs the value. Focus coverage efforts on **business logic and error handling that actually varies**.

## AI-Powered Coverage Improvement

AI excels at reading coverage reports and generating tests for uncovered paths. This is one of the highest-value uses of AI in testing.

### Workflow: Coverage Gap Analysis

```
1. Run tests with coverage:     npm test -- --coverage
2. Identify uncovered lines:    Read coverage report
3. Prompt AI with context:      "Generate tests for these uncovered branches"
4. Run tests again:             Verify coverage improved
5. Review AI tests:             Ensure they test meaningful behavior
```

### Prompt Template for Coverage Gaps

```
Here is the source file `src/order-service.ts`:

[paste source code]

Here are the uncovered branches from the coverage report:
- Line 45: else branch (when order total exceeds credit limit)
- Line 62: catch block (when payment gateway times out)
- Line 78: early return (when order has no items)
- Line 91-95: entire block (bulk discount calculation for 10+ items)

Generate tests that exercise each of these uncovered paths.
Use Jest with the existing mock setup pattern from the codebase.
Each test should verify the BEHAVIOR, not just execute the line.
```

### Example: Before and After

```typescript
// Source: order-service.ts (simplified)
async function processOrder(order: Order): Promise<OrderResult> {
  if (order.items.length === 0) {
    return { status: 'rejected', reason: 'Empty order' };        // ← uncovered
  }

  const total = calculateTotal(order.items);

  if (total > order.customer.creditLimit) {
    return { status: 'rejected', reason: 'Credit limit exceeded' }; // ← uncovered
  }

  try {
    await paymentGateway.charge(order.customer.id, total);
  } catch (error) {
    if (error instanceof TimeoutError) {
      return { status: 'pending', reason: 'Payment timeout' };     // ← uncovered
    }
    throw error;
  }

  return { status: 'confirmed', orderId: order.id };
}
```

AI-generated tests for uncovered paths:

```typescript
describe('processOrder — edge cases', () => {
  it('should reject order with no items', async () => {
    const order = createOrder({ items: [] });

    const result = await processOrder(order);

    expect(result).toEqual({
      status: 'rejected',
      reason: 'Empty order',
    });
    expect(paymentGateway.charge).not.toHaveBeenCalled();
  });

  it('should reject order exceeding credit limit', async () => {
    const order = createOrder({
      items: [{ price: 1000, quantity: 5 }], // total: 5000
      customer: { creditLimit: 2000 },
    });

    const result = await processOrder(order);

    expect(result).toEqual({
      status: 'rejected',
      reason: 'Credit limit exceeded',
    });
    expect(paymentGateway.charge).not.toHaveBeenCalled();
  });

  it('should return pending status on payment timeout', async () => {
    const order = createOrder({
      items: [{ price: 50, quantity: 1 }],
      customer: { creditLimit: 1000 },
    });
    paymentGateway.charge.mockRejectedValue(new TimeoutError('Gateway timeout'));

    const result = await processOrder(order);

    expect(result).toEqual({
      status: 'pending',
      reason: 'Payment timeout',
    });
  });

  it('should rethrow non-timeout payment errors', async () => {
    const order = createOrder({
      items: [{ price: 50, quantity: 1 }],
      customer: { creditLimit: 1000 },
    });
    paymentGateway.charge.mockRejectedValue(new Error('Network error'));

    await expect(processOrder(order)).rejects.toThrow('Network error');
  });
});
```

## Coverage Tools by Ecosystem

| Language | Tool | Command |
|----------|------|---------|
| JavaScript/TypeScript | [Istanbul/nyc](https://istanbul.js.org/) (via Jest) | `jest --coverage` |
| JavaScript/TypeScript | [c8](https://github.com/bcoe/c8) (V8 native) | `c8 npm test` |
| Python | [coverage.py](https://coverage.readthedocs.io/) | `coverage run -m pytest && coverage report` |
| Go | Built-in | `go test -coverprofile=coverage.out ./...` |
| Rust | [cargo-tarpaulin](https://github.com/xd009642/tarpaulin) | `cargo tarpaulin` |
| Java | [JaCoCo](https://www.jacoco.org/) | Maven/Gradle plugin |
| C# | [Coverlet](https://github.com/coverlet-coverage/coverlet) | `dotnet test --collect:"XPlat Code Coverage"` |

## When Coverage Metrics Lie

### High Coverage ≠ Correct Code

```typescript
// 100% coverage, 0% useful assertions
it('should process order', async () => {
  await processOrder(testOrder);
  // No assertions! We executed every line but verified nothing.
});
```

### Coverage Can Miss Logical Errors

```typescript
// This function has a bug: should be >= not >
function isAdult(age: number): boolean {
  return age > 18; // Bug: 18-year-olds are adults!
}

// This test gives 100% coverage but doesn't catch the bug
it('should identify adults', () => {
  expect(isAdult(25)).toBe(true);
  expect(isAdult(10)).toBe(false);
  // Missing: expect(isAdult(18)).toBe(true) ← would catch the bug
});
```

### Coverage Configuration for CI

```json
// jest.config.js — enforce coverage thresholds
{
  "coverageThreshold": {
    "global": {
      "branches": 80,
      "functions": 80,
      "lines": 80,
      "statements": 80
    },
    "./src/core/": {
      "branches": 90,
      "functions": 90,
      "lines": 90
    }
  }
}
```

## The Coverage Improvement Cycle

```
┌──────────────────────┐
│ Run tests + coverage │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ Identify low-coverage│    Focus on:
│ files and branches   │    - Business-critical code
└──────────┬───────────┘    - Recently changed files
           │                - Error handling paths
           ▼
┌──────────────────────┐
│ Prompt AI with source│
│ + uncovered branches │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ Review AI-generated  │    Check:
│ tests for quality    │    - Meaningful assertions?
└──────────┬───────────┘    - Testing behavior, not lines?
           │                - Edge cases covered?
           ▼
┌──────────────────────┐
│ Run tests again      │
│ Verify improvement   │
└──────────┴───────────┘
```

## Next Steps

- [Test Quality Metrics](./test-quality-metrics.md) — Go beyond coverage to measure if tests actually catch bugs
- [Mutation Testing](./mutation-testing.md) — Test whether your assertions are strong enough to detect code changes
- [AI-Generated Code Quality](./ai-code-quality.md) — Validate that AI code meets quality standards before it reaches production
