# Anti-Pattern: Over-Mocking

> When AI mocks so extensively that tests verify nothing — testing that mock functions were called instead of testing that real behavior works.

---

## The Problem

AI defaults to mocking every dependency because it makes tests easy to write and pass. But when everything is mocked, the test only verifies the **wiring** between mocks — not whether the actual system works.

```
What you think you're testing:      What you're actually testing:
──────────────────────────────      ──────────────────────────────
UserService → Database → Result     UserService → Mock → Mock Result
  "Create user saves to DB"           "Create user calls mock"
```

The test passes even if the database query is completely wrong, because the database was never involved.

---

## How AI Over-Mocks

### The Pattern

AI sees dependencies → mocks all of them → tests the glue code:

```javascript
// ❌ AI-generated: everything is mocked
const mockUserRepo = {
  create: jest.fn().mockResolvedValue({ id: 1, name: 'Alice' }),
  findByEmail: jest.fn().mockResolvedValue(null),
};
const mockEmailService = {
  sendWelcome: jest.fn().mockResolvedValue(true),
};
const mockLogger = {
  info: jest.fn(),
  error: jest.fn(),
};
const mockValidator = {
  validate: jest.fn().mockReturnValue({ valid: true }),
};

const service = new UserService(mockUserRepo, mockEmailService, mockLogger, mockValidator);

test('creates a user', async () => {
  await service.createUser({ name: 'Alice', email: 'alice@test.com' });

  expect(mockValidator.validate).toHaveBeenCalledWith({ name: 'Alice', email: 'alice@test.com' });
  expect(mockUserRepo.findByEmail).toHaveBeenCalledWith('alice@test.com');
  expect(mockUserRepo.create).toHaveBeenCalledWith({ name: 'Alice', email: 'alice@test.com' });
  expect(mockEmailService.sendWelcome).toHaveBeenCalledWith('alice@test.com');
  expect(mockLogger.info).toHaveBeenCalled();
});
```

### What This Test Actually Verifies

- ✅ `validate` was called with the right args — but was the validation logic correct? **Unknown**
- ✅ `findByEmail` was called — but does the query work? **Unknown**
- ✅ `create` was called — but does the SQL insert work? **Unknown**
- ✅ `sendWelcome` was called — but does the email send? **Unknown**

**This test would pass even if every single dependency was completely broken.**

---

## The Real-World Consequence

```javascript
// Real implementation has a bug:
class UserRepository {
  async create(userData) {
    // BUG: column name is 'user_name', not 'name'
    return this.db.query(
      'INSERT INTO users (name, email) VALUES ($1, $2)',  // Wrong column!
      [userData.name, userData.email]
    );
  }
}

// The over-mocked test? Still passes! ✅
// The mock doesn't know about column names.
// You discover this bug in production.
```

---

## Detection: How to Spot Over-Mocking

### Signal 1: More Mocks Than Assertions

```javascript
// Count the mocks vs meaningful assertions:
// Mocks: 4 objects with 6 mocked methods
// Assertions: all `toHaveBeenCalledWith` (verifying wiring, not behavior)
// Real behavior tested: ZERO
```

### Signal 2: Tests Pass When Implementation Is Broken

```javascript
// The acid test: break the implementation and see if tests catch it
class UserService {
  async createUser(data) {
    // Completely broken implementation
    return null;
  }
}

// If tests still pass → over-mocked
```

### Signal 3: No Integration Tests

If the test suite is 100% unit tests with extensive mocks and 0% integration tests, the gaps between components are completely untested.

### Signal 4: Mock Setup Longer Than Test

```javascript
// ❌ Red flag: 20 lines of mock setup, 5 lines of test
beforeEach(() => {
  mockRepo = { /* 8 methods mocked */ };
  mockService = { /* 5 methods mocked */ };
  mockLogger = { /* 3 methods mocked */ };
  mockCache = { /* 4 methods mocked */ };
  // ... continues for 20 more lines
});

test('does the thing', async () => {
  await doTheThing();
  expect(mockRepo.save).toHaveBeenCalled();
});
```

---

## The Fix: Strategic Mocking

### Rule: Mock at Boundaries, Not Everywhere

```
Your Code        Boundary        External
──────────       ────────        ────────
Service     →    Client     →    Stripe API     ← Mock HERE
Repository  →    Driver     →    PostgreSQL     ← Use test DB instead
Controller  →    Service    →    (internal)     ← Don't mock
```

**Mock external services** (APIs, email, payment). **Don't mock your own code**.

### Fix 1: Integration Tests with Real Dependencies

```javascript
// ✅ Integration test with real database
describe('UserService', () => {
  let service;
  let db;

  beforeAll(async () => {
    db = await createTestDatabase();
    const repo = new UserRepository(db);
    const emailService = new MockEmailService(); // Only mock external service
    service = new UserService(repo, emailService);
  });

  afterAll(async () => {
    await db.close();
  });

  beforeEach(async () => {
    await db.query('DELETE FROM users');
  });

  test('creates user in database', async () => {
    const user = await service.createUser({
      name: 'Alice',
      email: 'alice@test.com',
    });

    // Verify the REAL database has the user
    const saved = await db.query('SELECT * FROM users WHERE email = $1', ['alice@test.com']);
    expect(saved.rows).toHaveLength(1);
    expect(saved.rows[0].name).toBe('Alice');
  });

  test('rejects duplicate emails', async () => {
    await service.createUser({ name: 'Alice', email: 'alice@test.com' });

    await expect(
      service.createUser({ name: 'Bob', email: 'alice@test.com' })
    ).rejects.toThrow('Email already exists');
  });
});
```

### Fix 2: Limit Mocking Depth

```markdown
## Mocking Rules for AI

When generating tests:
- Mock ONLY external services (HTTP APIs, email, payment processors)
- Use a real test database (SQLite, Docker containers, or test schema)
- Never mock more than 1 layer deep
- If you need >3 mocks in a test, write an integration test instead
- Every mocked behavior should also be tested with a real integration test
```

### Fix 3: The Mocking Pyramid

```
                    ┌─────────────┐
                    │  E2E Tests  │  No mocks
                    │  (few)      │
                ┌───┴─────────────┴───┐
                │  Integration Tests  │  Mock only external
                │  (some)             │
            ┌───┴─────────────────────┴───┐
            │      Unit Tests             │  Mock direct dependencies
            │      (many)                 │  if truly needed
            └─────────────────────────────┘
```

---

## When Mocking IS Appropriate

| Scenario | Mock It? | Why |
|----------|---------|-----|
| External HTTP API | ✅ Yes | Unreliable, slow, may cost money |
| Email/SMS service | ✅ Yes | Don't send real emails in tests |
| Payment processor | ✅ Yes | Don't charge real cards |
| File system | ⚠️ Maybe | Use temp dirs if possible |
| Database | ❌ No | Use test database instead |
| Your own services | ❌ No | Test the real integration |
| Internal utilities | ❌ No | They should be fast enough to run real |

---

## Configuring AI to Avoid Over-Mocking

```markdown
# copilot-instructions.md or CLAUDE.md

## Mocking Rules
- Only mock external services (HTTP APIs, email, payment)
- Use real test database for data layer tests
- Never mock more than one dependency per test
- If a test needs >3 mocks, write an integration test instead
- Every mock should correspond to a real integration test elsewhere
- Prefer dependency injection over mocking
- Test behavior and output, not method calls
```

---

## Over-Mocking Detection Checklist

```markdown
## Code Review: Mock Audit
- [ ] Count mocks — more than 3 per test is a warning sign
- [ ] Assertions test behavior, not just "was called"
- [ ] Break the implementation — do tests still pass? (They shouldn't)
- [ ] External services are mocked, internal code is not
- [ ] Integration tests exist for every mocked boundary
- [ ] Mock return values are realistic (not just `true` or `{}`)
```
