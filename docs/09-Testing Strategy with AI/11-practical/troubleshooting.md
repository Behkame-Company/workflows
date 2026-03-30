# Troubleshooting AI Testing

> Common problems with AI-generated tests and how to fix them — from compilation errors to meaningless assertions.

---

## Problem 1: Tests Don't Compile

### Symptoms

```
SyntaxError: Cannot find module './UserService'
TypeError: UserService is not a constructor
ReferenceError: describe is not defined
```

### Root Cause

AI guesses import paths, framework APIs, or project structure. It may reference files that don't exist or use the wrong import syntax.

### Fix

```markdown
# Provide explicit context in your prompt:
Generate tests for `src/services/UserService.ts`.

Project info:
- Framework: Jest with ts-jest
- Import style: ES modules (`import { X } from './path'`)
- UserService is a class exported as named export
- Dependencies: UserRepository (constructor injection)
- Types are in `src/types/index.ts`
```

### Quick Fix Checklist

```markdown
- [ ] Check import paths — AI may use wrong relative paths
- [ ] Check framework imports — ensure correct test framework is imported
- [ ] Check TypeScript types — AI may use outdated type definitions
- [ ] Run `npx tsc --noEmit` to find type errors before running tests
```

---

## Problem 2: Tests Are Too Simple

### Symptoms

```javascript
// AI generates trivially obvious tests:
test('adds two numbers', () => {
  expect(add(1, 1)).toBe(2);
});

test('adds negative numbers', () => {
  expect(add(-1, 1)).toBe(0);
});
// That's it. No edge cases, no real scenarios.
```

### Root Cause

Prompt was too vague ("write tests for add function"). AI defaults to the simplest possible tests.

### Fix

Be specific about what to test:

```markdown
# Instead of: "Write tests for the add function"

Write tests for the add function that cover:
- Large numbers (Number.MAX_SAFE_INTEGER)
- Floating point precision (0.1 + 0.2)
- NaN inputs
- Infinity inputs
- Non-number inputs (strings, null, undefined, objects)
- Called with wrong number of arguments
- Performance with 1M sequential additions
```

### Prevention

Add to your instruction file:

```markdown
## Test Depth Requirements
- Minimum 3 happy path tests per function
- Minimum 2 edge case tests per function
- Minimum 2 error handling tests per function
- Test boundary values for all numeric inputs
- Test null/undefined for all optional parameters
```

---

## Problem 3: AI Modifies Tests to Pass

### Symptoms

```diff
# In AI's code change, you see test file modifications:
- expect(result.status).toBe(200);
+ expect(result.status).toBe(201);
```

### Root Cause

AI changes test expectations to match its implementation instead of fixing the implementation to match the test.

### Fix

```markdown
# Add to instruction file:
NEVER modify existing test files unless explicitly asked.
If existing tests fail after your changes, the implementation is wrong.
Fix the implementation, not the tests.
```

### Detection

```bash
# Check if AI modified test files alongside implementation:
git diff --name-only | grep -E "\.test\.|\.spec\." 

# If test files were modified in the same commit as implementation, investigate
```

---

## Problem 4: Flaky Test Generation

### Symptoms

Tests pass sometimes and fail others. Different results on CI vs local.

### Root Cause

AI generates tests with hidden non-determinism (see [Flaky Test Detection](../08-regression-and-maintenance/flaky-test-detection.md) for details).

### Quick Fix Table

| Flakiness Type | Detection | Fix |
|---------------|-----------|-----|
| Timing (`setTimeout`) | Search for `setTimeout` in tests | Replace with `waitFor` / `waitForExpect` |
| Shared state | Tests fail when run individually | Add `beforeEach` cleanup |
| Order dependency | Tests fail with `--randomize` | Isolate test data |
| Port conflicts | "Address in use" errors | Use dynamic ports |
| Date/time | Tests fail at month/year boundaries | Use fake timers |

### Validation

```bash
# Run tests 5 times to check for flakiness:
for i in {1..5}; do echo "Run $i:"; npm test -- --ci 2>&1 | tail -1; done
```

---

## Problem 5: Wrong Mocking Framework

### Symptoms

```
TypeError: jest.fn is not a function
Error: vi.mock is not a function
Cannot find module 'sinon'
```

### Root Cause

AI uses a different mocking library than your project.

### Fix

Specify explicitly:

```markdown
# In your prompt or instruction file:
## Mocking
- Mocking framework: Jest built-in (jest.fn(), jest.mock())
- Do NOT use sinon, testdouble, or vitest mocks
- HTTP mocking: msw (Mock Service Worker)
- Do NOT use nock or axios-mock-adapter
```

---

## Problem 6: Missing Imports

### Symptoms

```
ReferenceError: expect is not defined
ReferenceError: describe is not defined
Cannot find module '@testing-library/react'
```

### Root Cause

AI omits imports assuming they're global, or uses libraries not installed in the project.

### Fix

```markdown
# Tell AI about your test setup:
## Test Environment
- Jest globals are configured (describe, test, expect are global)
- Additional imports needed:
  - `import { render, screen } from '@testing-library/react'`
  - `import userEvent from '@testing-library/user-event'`
  - `import { rest } from 'msw'`
- Do NOT import from libraries not in package.json
```

### Quick Fix

```bash
# Find what testing libraries are installed:
cat package.json | python -c "
import json, sys
pkg = json.load(sys.stdin)
deps = {**pkg.get('dependencies', {}), **pkg.get('devDependencies', {})}
for name in sorted(deps):
    if any(kw in name for kw in ['test', 'jest', 'vitest', 'cypress', 'playwright', 'mock']):
        print(f'  {name}: {deps[name]}')
"
```

---

## Problem 7: Tests Pass But Verify Nothing

### Symptoms

Tests pass with 100% rate but mutation testing score is low. Tests don't catch intentional bugs.

### Root Cause

Tests are [testing-to-pass](../10-anti-patterns/testing-to-pass.md) or [over-mocked](../10-anti-patterns/mock-everything.md).

### Diagnostic

```bash
# The acid test: break the implementation
# 1. Introduce a bug in the function
# 2. Run tests
# 3. If all tests still pass → tests are worthless

# Or use mutation testing:
npx stryker run
```

### Fix

See the [Testing to Pass](../10-anti-patterns/testing-to-pass.md) and [Over-Mocking](../10-anti-patterns/mock-everything.md) anti-pattern guides.

---

## Debugging Workflow for AI Test Failures

When AI-generated tests fail and you're not sure why:

### Step 1: Isolate

```bash
# Run just the failing test:
npm test -- --testNamePattern="test name here"

# Run with verbose output:
npm test -- --verbose --testPathPattern="failing-test"
```

### Step 2: Understand

```markdown
# Ask AI to diagnose:
This test is failing with this error:

[paste error message]

The test code is:
[paste test]

The implementation is:
[paste implementation]

Explain why the test fails. Do NOT modify the test — tell me what's
wrong with the implementation.
```

### Step 3: Decide

| Situation | Action |
|-----------|--------|
| Test is correct, implementation is wrong | Fix implementation |
| Test has wrong assertion value | Check specification, update test if spec changed |
| Test setup is incomplete | Add proper setup/teardown |
| Test is fundamentally flawed | Rewrite test from specification |
| Test is testing implementation details | Rewrite to test behavior |

### Step 4: Verify

```bash
# After fix, run the specific test:
npm test -- --testPathPattern="fixed-test"

# Then run the full suite to check for regressions:
npm test
```

---

## When to Rewrite vs. Fix AI Tests

| Fix When... | Rewrite When... |
|------------|-----------------|
| Import paths are wrong | Test logic mirrors implementation |
| Setup/teardown is missing | Tests are over-mocked |
| Assertion values are slightly off | Tests don't test meaningful behavior |
| Async handling needs adjustment | Test structure is fundamentally wrong |
| Minor syntax errors | Tests are untestable (circular dependencies) |

---

## Quick Reference: Error → Fix

| Error Message | Likely Cause | Fix |
|--------------|-------------|-----|
| `Cannot find module` | Wrong import path | Check path, use IDE autocomplete |
| `X is not a function` | Wrong API usage | Check library version/docs |
| `Timeout exceeded` | Async not awaited | Add `await`, increase timeout |
| `Expected X, received Y` | Wrong assertion or bug | Check spec, fix implementation |
| `Cannot read property of undefined` | Missing setup | Add `beforeEach` initialization |
| `Address in use` | Port conflict | Use dynamic ports |
