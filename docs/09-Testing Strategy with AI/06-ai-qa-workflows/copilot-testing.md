# Testing with GitHub Copilot

> Using GitHub Copilot for test generation, verification, and continuous quality assurance across your development workflow.

---

## Overview

GitHub Copilot offers multiple surfaces for test generation — inline suggestions as you code, Chat commands for bulk generation, and the Coding Agent for autonomous test creation in pull requests. Understanding each surface helps you pick the right tool for each testing scenario.

## Copilot Inline Suggestions for Tests

The simplest way to generate tests is to start writing and let Copilot complete:

```typescript
// Start typing a test — Copilot suggests the rest
describe('UserService', () => {
  it('should create a user with valid input', async () => {
    // Copilot suggests: 
    const user = await userService.create({
      name: 'Alice',
      email: 'alice@example.com'
    });
    expect(user.id).toBeDefined();
    expect(user.name).toBe('Alice');
    expect(user.email).toBe('alice@example.com');
  });

  it('should reject duplicate email addresses', async () => {
    // Copilot continues the pattern...
  });
});
```

**Tips for better inline suggestions:**

- Write a descriptive `describe` block name — Copilot uses it as context
- Add a comment describing what the test should verify before accepting
- Write the first test manually to establish the pattern
- Keep the source file open in an adjacent tab for context

## Copilot Chat `/tests` Command

Use Copilot Chat to generate tests for existing code:

```
/tests Generate unit tests for the calculateDiscount function
```

Copilot Chat reads the selected code (or referenced file) and produces a complete test file. You can refine with follow-up prompts:

```
/tests Add edge cases: empty cart, negative prices, discount > 100%
```

```
/tests Generate integration tests for the OrderController using supertest
```

### Making `/tests` More Effective

| Technique | Example |
|-----------|---------|
| Specify framework | `/tests using Jest and React Testing Library` |
| Request edge cases | `/tests include boundary conditions and error paths` |
| Reference patterns | `/tests follow the pattern in user.test.ts` |
| Target coverage | `/tests cover all branches in the switch statement` |

## Copilot Coding Agent: Test Configuration

The Copilot Coding Agent can autonomously write and run tests when working on pull requests. Configure it through your repository's instruction files.

### copilot-instructions.md — Test Sections

Add test-specific instructions so the agent knows how to run and write tests:

```markdown
<!-- .github/copilot-instructions.md -->

## Testing

### General Test Guidelines
- Always run tests after making code changes
- Use `npm test` for unit tests
- Use `npm run test:integration` for integration tests
- Test files live next to source files with `.test.ts` suffix
- Use Jest with TypeScript for all tests
- Mock external services — never call real APIs in tests

### Path-Specific Test Instructions

For files in `src/api/**`:
- Run: `npm run test -- --testPathPattern=src/api`
- Include request/response validation tests
- Test authentication and authorization separately

For files in `tests/e2e/**`:
- Run: `npx playwright test`
- Ensure the dev server is running first: `npm run dev &`
- Use Page Object Model pattern
- Test against both Chrome and Firefox

For files in `src/utils/**`:
- Run: `npm test -- --testPathPattern=src/utils`
- Achieve 100% branch coverage for utility functions
- Include property-based tests for pure functions

### Test Quality Requirements
- Each test must have a descriptive name explaining the scenario
- Use Arrange-Act-Assert pattern
- No test should depend on another test's state
- Clean up test data in afterEach hooks
```

### copilot-setup-steps.yml — Pre-Installing Test Dependencies

Ensure the agent's environment has everything needed to run tests:

```yaml
# .github/copilot-setup-steps.yml
steps:
  - uses: actions/checkout@v4

  - uses: actions/setup-node@v4
    with:
      node-version: '20'
      cache: 'npm'

  - run: npm ci

  # Install browsers for Playwright E2E tests
  - run: npx playwright install --with-deps chromium firefox

  # Start services needed for integration tests
  - run: docker compose -f docker-compose.test.yml up -d

  # Wait for services to be ready
  - run: npm run wait-for-services

  # Seed test database
  - run: npm run db:seed:test
```

⚠️ **Important:** The setup steps run before the agent starts working. If the agent can't run tests, it can't verify its changes — leading to broken PRs.

## Tips for Better Copilot Test Generation

### 1. Provide Context Through Open Files

Copilot uses open files as context. When generating tests:

- Open the source file being tested
- Open an existing test file that shows your patterns
- Open type definitions or interfaces used by the code

### 2. Use Descriptive Function and Type Names

```typescript
// ❌ Vague — Copilot guesses at behavior
function process(d: any): any { ... }

// ✅ Clear — Copilot understands intent
function validateEmailFormat(email: string): ValidationResult { ... }
```

### 3. Write the Test Structure, Let Copilot Fill Details

```typescript
describe('PaymentProcessor', () => {
  describe('processPayment', () => {
    it('should charge the correct amount for a standard order');
    it('should apply discount codes before charging');
    it('should reject expired credit cards');
    it('should handle network timeouts gracefully');
    it('should not charge twice on retry');
    // Let Copilot implement each test body
  });
});
```

### 4. Prompt for Specific Scenarios

Instead of generic requests, be specific:

```
// Generate a test for when the API returns a 429 rate limit response
// and the client should retry with exponential backoff
```

### 5. Always Review Generated Tests

Copilot-generated tests can have issues:

| Issue | What to Check |
|-------|---------------|
| **Tautological tests** | Does the test actually verify behavior, or just repeat the implementation? |
| **Missing assertions** | Does every test have meaningful `expect` statements? |
| **Hardcoded values** | Are magic numbers explained or extracted to constants? |
| **Incomplete mocking** | Are external dependencies properly isolated? |
| **Happy path only** | Are error cases and edge cases covered? |

## Verification Workflow

```
┌─────────────────────┐
│  Write/modify code   │
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  Copilot generates   │
│  or updates tests    │
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  Run test suite      │──── Failures? ──→ Fix and re-run
└─────────┬───────────┘
          │ Pass
          ▼
┌─────────────────────┐
│  Review test quality │──── Gaps? ──→ Request more tests
└─────────┬───────────┘
          │ Good
          ▼
┌─────────────────────┐
│  Commit with         │
│  confidence          │
└─────────────────────┘
```
