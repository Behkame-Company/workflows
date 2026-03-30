# Test Generation Prompts

> Writing effective prompts for comprehensive test creation.

---

## The Test Generation Formula

```
[What to test] + [Test framework] + [Coverage expectations] + [Edge cases]
```

---

## Basic Test Generation

### Minimum Viable Prompt
```
Write Jest tests for the createUser function in src/services/user.ts.
Use TypeScript. Mock the database with jest.mock.
Cover: happy path, validation errors, duplicate email, database failure.
```

### Comprehensive Prompt
```
Write comprehensive Jest tests for src/services/user.ts :: createUser()

Framework: Jest + ts-jest
Mocking: Use jest.mock for @prisma/client
Test file: src/services/__tests__/user.test.ts

Test categories:
1. Happy path: Valid user creation, returns user object
2. Validation: Invalid email, short password (<8 chars), missing required fields
3. Conflicts: Duplicate email returns specific error
4. Dependencies: Database connection failure, timeout
5. Edge cases: Very long name (255 chars), unicode in name, whitespace email

For each test:
- Clear test name describing the scenario
- Arrange → Act → Assert pattern
- Specific assertion (not just "doesn't throw")

Do NOT test implementation details (private methods, internal state).
Only test the public interface.
```

---

## Test-First / TDD Prompts

### Write Tests Before Implementation
```
I'm about to implement a rate limiter. Write the tests first.

Function signature:
  rateLimit(userId: string, action: string): Promise<{ allowed: boolean; retryAfter?: number }>

Rules:
- 100 requests per minute per user per action
- Returns { allowed: false, retryAfter: seconds } when exceeded
- Time window slides (not fixed windows)

Write tests that I'll make pass one by one. Order from simplest to most complex.
```

### Red-Green-Refactor Guidance
```
Here's a failing test:
[test code]

Write the MINIMUM code to make this test pass.
Do not add extra functionality. Do not handle cases 
not covered by existing tests.
```

---

## Structured Test Output

### JSON Test List (for tracking)

Anthropic recommends creating tests in structured format:

```
Create a test plan as JSON for the checkout flow:

{
  "tests": [
    {
      "id": "checkout-001",
      "category": "happy-path",
      "description": "User can complete checkout with valid credit card",
      "status": "not_written",
      "priority": "critical"
    },
    ...
  ]
}
```

This format enables tracking across multiple sessions.

---

## Avoiding Test Anti-Patterns

Tell the model what to avoid:

```
When writing tests:
- Do NOT test implementation details (which functions are called internally)
- Do NOT use test.skip or pending tests
- Do NOT share state between tests (each test should be independent)
- Do NOT assert on exact error messages (assert on error types/codes)
- Do NOT create brittle snapshot tests for dynamic data
- Do NOT mock what you don't own — wrap external APIs in adapters

Each test should follow: Arrange → Act → Assert
Each test should have ONE assertion concept (may use multiple expect() calls)
```

---

## Testing Prompts by Type

| Test Type | Key Prompt Elements |
|-----------|-------------------|
| Unit test | Mock dependencies, test function in isolation |
| Integration test | "Do NOT mock [service], test the real interaction" |
| E2E test | Describe the user flow step by step |
| Property-based | "Generate random inputs that satisfy [constraints]" |
| Regression test | "Add a test that catches [specific bug] to prevent recurrence" |
