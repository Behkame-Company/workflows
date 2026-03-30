# Test-First Development with AI

> Making test-first the default workflow for all AI-assisted coding — because writing tests before code transforms AI from a guessing machine into a specification executor.

---

## The Culture Shift

Test-first with AI is fundamentally different from test-first without AI:

```
Traditional TDD:                    AI-Assisted Test-First:
────────────────                    ───────────────────────
Human writes test                   Human writes test
Human writes code to pass test      AI writes code to pass test
Human refactors                     Human reviews, AI refactors

Bottleneck: writing code            Bottleneck: writing good tests
```

The key insight: **with AI, tests are your primary creative output**. The code is generated. Your tests are what define quality.

---

## Why Test-First Is Dramatically Better with AI

### Without Test-First (Test-After)

```markdown
1. Ask AI: "Create a user registration function"
2. AI generates code (plausible but unverified)
3. You look at it, think "looks right"
4. Ask AI: "Now write tests for it"
5. AI writes tests that match its own code ← CIRCULAR!
6. 100% pass rate, 0% confidence
```

### With Test-First

```markdown
1. Write tests that define correct behavior
2. Run tests → all fail (RED) ← Proves tests actually check something
3. Ask AI: "Make these tests pass"
4. AI generates code constrained by YOUR tests
5. Run tests → pass (GREEN) ← Real verification
6. Review code, refactor with AI assistance
```

### The Benefits

| Benefit | Why It Matters |
|---------|---------------|
| **Clearer requirements** | Tests force you to define exact expected behavior |
| **Verifiable output** | You know immediately if AI code is correct |
| **Faster feedback** | Seconds to verify vs minutes of code review |
| **Specification lock** | AI can't drift from requirements — tests enforce them |
| **Regression protection** | Tests exist before the code, not as an afterthought |

---

## The Test-First Workflow

### Step 1: Human Writes Tests

```javascript
// You write this — it defines the specification
describe('PasswordValidator', () => {
  test('accepts passwords 8+ characters with uppercase, lowercase, and number', () => {
    expect(validatePassword('Secure1234')).toEqual({ valid: true });
  });

  test('rejects passwords shorter than 8 characters', () => {
    expect(validatePassword('Short1')).toEqual({
      valid: false,
      reason: 'Password must be at least 8 characters',
    });
  });

  test('rejects passwords without uppercase letter', () => {
    expect(validatePassword('nouppercase1')).toEqual({
      valid: false,
      reason: 'Password must contain at least one uppercase letter',
    });
  });

  test('rejects passwords without a number', () => {
    expect(validatePassword('NoNumberHere')).toEqual({
      valid: false,
      reason: 'Password must contain at least one number',
    });
  });

  test('rejects empty string', () => {
    expect(validatePassword('')).toEqual({
      valid: false,
      reason: 'Password must be at least 8 characters',
    });
  });
});
```

### Step 2: Verify Tests Fail (RED)

```bash
$ npm test -- --testPathPattern="password"

FAIL  tests/passwordValidator.test.js
  ✕ accepts passwords 8+ characters with uppercase, lowercase, and number
  ✕ rejects passwords shorter than 8 characters
  ✕ rejects passwords without uppercase letter
  ✕ rejects passwords without a number
  ✕ rejects empty string

Tests: 5 failed, 5 total
```

All failing — good! This proves the tests actually check something.

### Step 3: AI Implements (GREEN)

```markdown
# Prompt to AI
Implement the `validatePassword` function that makes all tests in
tests/passwordValidator.test.js pass.

Requirements are defined entirely by the tests. Do not add behavior
beyond what the tests specify. Run the tests after implementing.
```

### Step 4: Run Tests → Verify

```bash
$ npm test -- --testPathPattern="password"

PASS  tests/passwordValidator.test.js
  ✓ accepts passwords 8+ characters with uppercase, lowercase, and number
  ✓ rejects passwords shorter than 8 characters
  ✓ rejects passwords without uppercase letter
  ✓ rejects passwords without a number
  ✓ rejects empty string

Tests: 5 passed, 5 total
```

### Step 5: Review and Refactor

Review the AI-generated implementation for:
- Security issues the tests don't cover
- Performance concerns
- Code clarity
- Additional edge cases to add

---

## When Test-First Doesn't Work

Test-first isn't always practical. Recognize these situations:

| Situation | Why Test-First Is Hard | Alternative |
|-----------|----------------------|-------------|
| **Exploratory coding** | You don't know what the output looks like | Spike first, then write tests for what works |
| **UI prototyping** | Visual output is subjective | Build prototype, then add interaction tests |
| **Research/learning** | Requirements are undefined | Experiment first, formalize with tests later |
| **Data pipeline exploration** | You need to see the data first | Explore data, then test transformations |

### The Hybrid Approach

```markdown
1. Explore briefly (15-30 minutes max)
2. Once you understand the shape of the solution, STOP
3. Write tests for what you discovered
4. Ask AI to implement to the tests
```

---

## Making Test-First Easy

### Test Templates

```javascript
// Template: Unit test for a pure function
describe('[FunctionName]', () => {
  describe('valid inputs', () => {
    test('[describes expected behavior for typical input]', () => {
      expect(functionName(/* typical input */)).toEqual(/* expected output */);
    });
  });

  describe('edge cases', () => {
    test('handles empty input', () => {
      expect(functionName(/* empty */)).toEqual(/* expected */);
    });

    test('handles boundary values', () => {
      expect(functionName(/* min/max */)).toEqual(/* expected */);
    });
  });

  describe('error cases', () => {
    test('rejects invalid input', () => {
      expect(() => functionName(/* invalid */)).toThrow(/* expected error */);
    });
  });
});
```

### Prompt Patterns for Test-First

```markdown
# When asking AI to implement:
I've written failing tests in [file]. Implement the [function/class] to make
all tests pass.

Rules:
- Only implement what the tests require
- Do not modify the test file
- Run tests after implementing
- If a test seems wrong, tell me instead of changing it
```

---

## Test-First Checklist for PRs

```markdown
## PR Review: Test-First Verification
- [ ] Tests were committed BEFORE implementation (check git log)
- [ ] Tests fail without the implementation (verified in CI or locally)
- [ ] No test files were modified in implementation commits
- [ ] Tests cover the requirements, not just the implementation
- [ ] Edge cases are tested
- [ ] Error cases are tested
- [ ] AI was not asked to "write tests for this code" (test-after pattern)
```

### Git Log Verification

```bash
# Verify test-first commit order
git log --oneline --name-only | head -20

# Should show:
# abc1234 feat: implement password validation
#   src/passwordValidator.js
# def5678 test: add password validation tests
#   tests/passwordValidator.test.js
#
# Tests committed BEFORE implementation ✅
```

---

## Configuring AI Tools for Test-First

```markdown
# copilot-instructions.md
## Development Workflow
- Always expect tests to exist before implementing features
- If tests don't exist for a feature request, ask for them first
- Never modify test files when implementing features
- Run tests after every implementation change
```

```markdown
# CLAUDE.md
## Workflow
- Follow test-first development: tests are written before implementation
- When given a task without tests, write tests first, show them, then implement
- Never change test expectations to match implementation
- Run the full test suite after any change
```
