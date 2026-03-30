# Test Quality Metrics

> Beyond line coverage — measuring whether your tests actually catch bugs. High coverage with weak assertions gives false confidence. These metrics reveal the real strength of your test suite.

## Why Coverage Isn't Enough

```typescript
// 100% line coverage — but catches ZERO bugs
describe('calculateDiscount', () => {
  it('runs without error', () => {
    const result = calculateDiscount(100, 'SAVE20');
    expect(result).toBeDefined(); // Passes even if result is wrong
  });
});
```

Coverage tells you code was **executed**. Quality metrics tell you code was **verified**.

## Key Test Quality Metrics

### 1. Assertion Density

**What**: Average number of meaningful assertions per test case.

**Why it matters**: AI-generated tests often have LOW assertion density — one `expect` per test, frequently just checking that a function "doesn't throw."

| Rating | Assertions per Test | Notes |
|--------|-------------------|-------|
| 🔴 Weak | < 1.5 | Likely testing execution, not behavior |
| 🟡 Acceptable | 1.5 – 3.0 | Reasonable for most tests |
| 🟢 Strong | 3.0+ | Thorough verification of behavior |

```typescript
// 🔴 Assertion density: 1.0
it('should create user', async () => {
  const user = await service.createUser(validInput);
  expect(user).toBeDefined();
});

// 🟢 Assertion density: 4.0
it('should create user with correct properties and side effects', async () => {
  const user = await service.createUser(validInput);
  expect(user.email).toBe('alice@example.com');
  expect(user.passwordHash).not.toBe(validInput.password); // hashed
  expect(user.createdAt).toBeInstanceOf(Date);
  expect(mockEmailService.sendWelcome).toHaveBeenCalledWith('alice@example.com');
});
```

### 2. Mutation Score

**What**: Percentage of code mutations caught by your tests. See [Mutation Testing](./mutation-testing.md) for details.

**Target**: 70%+ for critical code, 50%+ for general code.

### 3. Test Independence

**What**: Whether each test can run in isolation, in any order, without affecting other tests.

**How to check**:
```bash
# Run tests in random order (Jest)
jest --randomize

# Run a single test file in isolation
jest path/to/specific.test.ts
```

**Common independence violations**:
- Tests that share mutable state
- Tests that depend on execution order
- Tests that don't clean up database or file system changes

### 4. Execution Time

**What**: How long individual tests and the full suite take to run.

| Rating | Unit Test | Integration Test | Full Suite |
|--------|-----------|-----------------|------------|
| 🟢 Fast | < 50ms | < 2s | < 2 min |
| 🟡 Acceptable | 50-200ms | 2-10s | 2-10 min |
| 🔴 Slow | > 200ms | > 10s | > 10 min |

Slow tests discourage developers from running them. AI-generated tests sometimes import heavy modules or create unnecessary async operations.

### 5. Failure Diagnostics Quality

**What**: When a test fails, how quickly can you identify the problem?

```typescript
// 🔴 Bad failure message:
//   "Expected false to be true"
expect(isValid).toBe(true);

// 🟢 Good failure message:
//   "Expected email 'not-an-email' to pass validation, but it was rejected"
expect(validateEmail(email)).toBe(true);
// or better:
if (!validateEmail(email)) {
  throw new Error(`Expected email '${email}' to pass validation`);
}

// 🟢 Jest custom matchers for clear messages:
expect(result).toEqual(
  expect.objectContaining({
    status: 'confirmed',
    orderId: expect.any(String),
  })
);
// Failure: "Expected object containing {status: 'confirmed'} but received {status: 'rejected', reason: 'Insufficient funds'}"
```

### 6. Behavior Coverage Ratio

**What**: Ratio of documented/expected behaviors that have corresponding tests.

```
Feature: User Registration
  Behaviors:
    ✅ Create account with valid email and password
    ✅ Reject duplicate email
    ✅ Reject weak password
    ❌ Send welcome email after registration     ← untested behavior
    ❌ Rate limit registration attempts           ← untested behavior
    ✅ Hash password before storing

  Behavior coverage: 4/6 = 67%
```

This is different from line coverage — you can have 100% line coverage and still miss testing important behaviors.

## AI-Generated Test Quality Problems

AI tools tend to produce tests with predictable quality gaps:

| Problem | Frequency | Example |
|---------|-----------|---------|
| Single weak assertion | Very common | `expect(result).toBeDefined()` |
| Happy path only | Very common | No error case testing |
| Testing implementation details | Common | Checking internal method calls |
| Copy-paste test structure | Common | Same test with slightly different inputs |
| Missing edge cases | Common | No null, empty, boundary checks |
| Excessive mocking | Occasional | Mocking the thing being tested |

### How to Prompt for Assertion-Rich Tests

```
Write tests for OrderService.calculateTotal().

Requirements for EVERY test:
- At least 3 assertions per test
- Test the return value AND any side effects
- Include negative assertions (what should NOT happen)
- Verify edge cases: empty arrays, single items, max values

Do NOT write tests that only check:
- .toBeDefined()
- .toBeTruthy()
- .not.toThrow()
These are too weak. Check actual values and behavior.
```

## Test Quality Checklist

Use this checklist to evaluate any test — human or AI written:

```markdown
## Test Quality Review

### Per-Test Checks
- [ ] Test name describes the behavior being verified
- [ ] Uses AAA (Arrange-Act-Assert) structure
- [ ] Has 2+ meaningful assertions
- [ ] Tests behavior, not implementation
- [ ] No shared mutable state with other tests
- [ ] Failure message clearly identifies what went wrong

### Per-Suite Checks
- [ ] Covers happy path AND error paths
- [ ] Includes edge cases (null, empty, boundary values)
- [ ] Tests run independently in any order
- [ ] No flaky tests (passes consistently on repeat runs)
- [ ] Setup/teardown is clean — no leaking state
- [ ] Execution time is reasonable for test type

### AI-Specific Checks
- [ ] Not just testing that code "runs without error"
- [ ] Assertions check actual values, not just type/existence
- [ ] Doesn't over-mock (mocking the thing being tested)
- [ ] Tests would fail if a real bug were introduced
- [ ] Follows project conventions, not generic patterns
```

## Metrics Summary Table

| Metric | What It Measures | Target | How to Check |
|--------|-----------------|--------|--------------|
| Line coverage | Code executed | ≥ 80% | `jest --coverage` |
| Branch coverage | Conditionals tested | ≥ 80% | Coverage report |
| Assertion density | Verification depth | ≥ 2.0 per test | Manual review or custom lint rule |
| Mutation score | Assertion strength | ≥ 70% critical | Stryker / mutmut |
| Test independence | Isolation | 100% | `jest --randomize` |
| Execution time | Developer feedback speed | < 2 min (unit) | CI metrics |
| Failure diagnostics | Debug speed | Clear messages | Review on failure |
| Behavior coverage | Feature completeness | ≥ 90% critical paths | Manual audit against requirements |

## Automating Quality Checks

```yaml
# .github/workflows/test-quality.yml
name: Test Quality Gate
on: [pull_request]

jobs:
  test-quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm test -- --coverage --randomize
      - name: Check coverage thresholds
        run: |
          # Jest will fail if thresholds in jest.config.js aren't met
          npm test -- --coverage --ci
      - name: Run mutation testing on changed files
        run: |
          npx stryker run --mutate "src/**/*.ts" --reporters clear-text
```
