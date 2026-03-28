# Test Maintenance

> Keeping tests healthy, meaningful, and well-organized as AI evolves your codebase — because unmaintained tests become liabilities, not assets.

---

## Why Test Maintenance Matters More with AI

AI accelerates code changes, which means tests age faster:

- **Rapid API evolution** — AI refactors can change interfaces frequently
- **Test volume explosion** — AI generates many tests, but not all stay relevant
- **Test rot accumulation** — unused or broken tests pile up when AI rewrites code
- **Inconsistent styles** — different AI sessions produce tests with different conventions

```
Without Maintenance:          With Maintenance:
──────────────────────        ─────────────────
Week 1:  50 tests ✅          Week 1:  50 tests ✅
Week 4: 120 tests, 8 ⚠️       Week 4: 110 tests ✅
Week 8: 200 tests, 30 ❌      Week 8: 180 tests ✅
Week 12: Nobody trusts         Week 12: Team trusts
         the test suite                  the test suite
```

---

## Identifying Test Rot

Test rot occurs when tests become unreliable, meaningless, or misleading:

### Types of Test Rot

| Type | Symptoms | Example |
|------|----------|---------|
| **Stale assertions** | Tests pass but verify outdated behavior | Testing old API response format |
| **Dead tests** | Tests that never run or are permanently skipped | `test.skip('TODO: fix later')` |
| **Tautological tests** | Tests that can't fail | `expect(true).toBe(true)` after a refactor |
| **Orphaned tests** | Tests for deleted features | Testing a removed endpoint |
| **Fragile tests** | Break on any change, even correct ones | Hardcoded timestamps |

### Detection Script

```bash
# Find skipped tests
grep -rn "test.skip\|xit(\|xdescribe(\|@Disabled\|@pytest.mark.skip" tests/

# Find tests with no assertions
grep -rL "expect\|assert\|should" tests/**/*.test.*

# Find test files not modified in 6+ months
find tests/ -name "*.test.*" -mtime +180

# Find TODO/FIXME in tests
grep -rn "TODO\|FIXME\|HACK\|XXX" tests/
```

---

## Refactoring Tests Alongside Code

When AI refactors production code, tests must evolve too:

### Pattern: Synchronized Refactoring

```markdown
# Prompt for AI-assisted refactoring
Refactor the UserService class to use the Repository pattern.

Requirements:
1. Move database logic from UserService to UserRepository
2. Update ALL existing tests to reflect the new structure
3. Maintain the same test coverage
4. Do NOT delete any test cases — move them to appropriate test files
5. Run all tests after refactoring to confirm they pass
```

### Example: API Change

```javascript
// BEFORE: UserService had direct DB access
// tests/userService.test.js
describe('UserService', () => {
  test('creates user in database', async () => {
    const user = await userService.create({ name: 'Alice' });
    expect(user.id).toBeDefined();
  });
});

// AFTER: Refactored to Repository pattern
// tests/userService.test.js — updated
describe('UserService', () => {
  test('creates user via repository', async () => {
    const mockRepo = { create: jest.fn().mockResolvedValue({ id: 1, name: 'Alice' }) };
    const service = new UserService(mockRepo);
    const user = await service.create({ name: 'Alice' });
    expect(mockRepo.create).toHaveBeenCalledWith({ name: 'Alice' });
    expect(user.id).toBeDefined();
  });
});

// tests/userRepository.test.js — new integration test
describe('UserRepository', () => {
  test('creates user in database', async () => {
    const repo = new UserRepository(testDb);
    const user = await repo.create({ name: 'Alice' });
    expect(user.id).toBeDefined();
  });
});
```

---

## Using AI to Update Tests When APIs Change

AI excels at bulk test updates when interfaces change:

```markdown
# Effective prompt for test updates
The `createUser` function signature changed:

Before: createUser(name: string, email: string)
After:  createUser(params: { name: string, email: string, role?: string })

Update all test files that call createUser to use the new signature.
Do NOT change what the tests verify — only update how they call the function.
Show me the diff for each file.
```

### Batch Update Pattern

```javascript
// AI can update dozens of test files consistently:

// Before (across 15 test files):
const user = await createUser('Alice', 'alice@example.com');

// After (AI updates all 15 files):
const user = await createUser({ name: 'Alice', email: 'alice@example.com' });
```

---

## Removing Obsolete Tests

Not every test should live forever. Remove tests that:

### Removal Criteria

```markdown
## Test Removal Checklist
- [ ] Feature was intentionally deleted (not just refactored)
- [ ] Test is a duplicate of another test (identical behavior verified)
- [ ] Test verifies implementation details that changed (not behavior)
- [ ] Test has been skipped for 30+ days with no fix plan
- [ ] Test is tautological (can never fail)
```

### Safe Removal Process

```bash
# 1. Identify candidate tests
grep -rn "test.skip\|@Disabled" tests/ > candidates.txt

# 2. Check when they were last meaningful
git log --oneline -5 -- tests/obsolete-feature.test.js

# 3. Verify no unique coverage is lost
npm test -- --coverage --collectCoverageFrom='src/**'
# Compare coverage before and after removal

# 4. Remove with a clear commit message
git rm tests/obsolete-feature.test.js
git commit -m "test: remove obsolete tests for deleted export feature

Feature removed in #342. Tests verified behavior that no longer exists.
Coverage unchanged — remaining tests cover current functionality."
```

---

## Test Code Review Standards

Review AI-generated tests with the same rigor as production code:

### Review Checklist

```markdown
## AI-Generated Test Review
- [ ] **Meaningful assertions** — tests verify behavior, not implementation
- [ ] **Readable names** — test names describe the scenario, not the code
- [ ] **No logic duplication** — tests don't copy production logic into assertions
- [ ] **Proper setup/teardown** — no shared state between tests
- [ ] **Edge cases covered** — not just the happy path
- [ ] **Error scenarios** — tests verify failure modes
- [ ] **No hardcoded values** — timestamps, IDs, ports are dynamic
- [ ] **Consistent style** — matches team conventions
```

### Naming Conventions

```javascript
// ❌ Bad: AI-generated names that describe code
test('should return true when isValid is called with valid input', ...);
test('test createUser function', ...);
test('userService test 1', ...);

// ✅ Good: Names that describe behavior
test('accepts emails with standard format (user@domain.com)', ...);
test('rejects passwords shorter than 8 characters', ...);
test('returns 404 when user does not exist', ...);
```

---

## Test Documentation

Keep tests self-documenting:

```javascript
/**
 * Order Processing Tests
 *
 * These tests verify the complete order lifecycle:
 * 1. Order creation with validation
 * 2. Payment processing
 * 3. Inventory updates
 * 4. Notification dispatch
 *
 * Dependencies: test database, mock payment gateway
 * Setup: see beforeAll() for test data seeding
 */
describe('Order Processing', () => {
  // Group related tests
  describe('creation', () => {
    test('creates order with valid items and shipping address', ...);
    test('rejects order with empty cart', ...);
    test('rejects order with invalid shipping address', ...);
  });

  describe('payment', () => {
    test('processes valid credit card payment', ...);
    test('handles declined card gracefully', ...);
    test('retries on transient payment gateway errors', ...);
  });
});
```

---

## Test Organization Patterns

### By Feature (Recommended)

```
tests/
├── auth/
│   ├── login.test.js
│   ├── registration.test.js
│   └── password-reset.test.js
├── orders/
│   ├── create-order.test.js
│   ├── process-payment.test.js
│   └── order-history.test.js
└── shared/
    ├── test-helpers.js
    ├── fixtures/
    └── factories/
```

### By Test Type

```
tests/
├── unit/
│   ├── services/
│   └── utils/
├── integration/
│   ├── api/
│   └── database/
├── e2e/
│   ├── user-flows/
│   └── admin-flows/
└── shared/
    ├── fixtures/
    └── helpers/
```

### Maintenance Schedule

```markdown
## Monthly Test Health Review
1. Run full suite — note any failures or skips
2. Check test execution time — flag tests slower than 5s
3. Review coverage report — identify declining areas
4. Audit skipped tests — remove or fix
5. Check for test rot patterns (stale assertions, dead code)
6. Update test documentation if features changed
7. Ensure AI instruction files reference current test patterns
```

---

## Next Steps

- [Regression Testing Strategy](regression-testing.md) — preventing AI-introduced regressions
- [Flaky Test Detection](flaky-test-detection.md) — identifying non-deterministic tests
- [Testing Templates](../11-practical/templates.md) — ready-to-use test maintenance templates
- [Structured Test Tracking](../09-best-practices/structured-test-tracking.md) — tracking test state across sessions

---

*Part of the [Testing Strategy with AI](../README.md) series.*
