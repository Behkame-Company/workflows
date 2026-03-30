# Testing with Claude Code

> Claude Code testing workflows — from interactive test-driven development to CI pipeline integration with structured verification.

---

## Overview

Claude Code operates as a terminal-based AI agent that can write, run, and iterate on tests directly in your development environment. Its agentic workflow — where it executes commands, reads output, and self-corrects — makes it particularly effective for test-driven development and verification loops.

## CLAUDE.md Test Instructions

The `CLAUDE.md` file is Claude Code's primary instruction source. Add test configuration so the agent always knows how to run and validate tests:

```markdown
<!-- CLAUDE.md -->

# Project: E-Commerce API

## Test Commands
- Unit tests: `npm test`
- Integration tests: `npm run test:integration`
- E2E tests: `npm run test:e2e`
- Single file: `npm test -- --testPathPattern=<filename>`
- Coverage: `npm run test:coverage`

## Test Conventions
- Test files: `*.test.ts` next to source files
- Use Jest + supertest for API tests
- Use React Testing Library for component tests
- Mock external APIs with `msw` (Mock Service Worker)
- Database tests use a dedicated test database: `myapp_test`

## Test Requirements
- Run relevant tests after EVERY code change
- All tests must pass before considering a task complete
- New features require unit tests at minimum
- Bug fixes require a regression test proving the fix
- Never modify existing tests to make them pass — fix the code instead

## Test Patterns
- Arrange-Act-Assert structure
- One assertion concept per test
- Descriptive test names: "should [expected behavior] when [condition]"
- Use factories for test data, not inline objects
```

### Directory-Specific CLAUDE.md

Place additional `CLAUDE.md` files in subdirectories for path-specific instructions:

```markdown
<!-- tests/e2e/CLAUDE.md -->

## E2E Test Instructions
- Start dev server before running: `npm run dev &`
- Wait for server: `curl --retry 10 --retry-delay 1 http://localhost:3000/health`
- Run with: `npx playwright test`
- Use Page Object Model — see `tests/e2e/pages/` for examples
- Screenshots on failure go to `tests/e2e/screenshots/`
- Always clean up test data after each test suite
```

## Multi-Window Testing Workflow

Claude Code's most powerful testing pattern uses two terminal windows working in tandem:

```
┌──────────────────────────┐    ┌──────────────────────────┐
│     Window 1: Tests      │    │   Window 2: Implementation│
│                          │    │                          │
│  "Write failing tests    │    │  "Make these tests pass  │
│   for user registration  │───▶│   by implementing the    │
│   with email validation" │    │   registration service"  │
│                          │    │                          │
│  Writes test suite       │    │  Reads tests, implements │
│  Runs → all fail (red)   │    │  Runs → iterates until   │
│                          │    │  all green               │
└──────────────────────────┘    └──────────────────────────┘
```

### Step-by-Step

1. **Window 1** — Write the test specification:
   ```
   > Write comprehensive tests for a UserRegistration service that:
   > - Validates email format
   > - Checks password strength (min 8 chars, uppercase, number)
   > - Prevents duplicate emails
   > - Sends welcome email on success
   > - Returns appropriate error messages
   > Run the tests to confirm they all fail.
   ```

2. **Window 2** — Implement against the tests:
   ```
   > Read the tests in src/services/user-registration.test.ts
   > Implement UserRegistrationService to make all tests pass.
   > Run the tests after each significant change.
   ```

This separation enforces true TDD — the tests define the contract before implementation begins.

## CI Integration with `claude --print`

Use `claude --print` (non-interactive mode) for CI pipeline integration:

```yaml
# .github/workflows/ai-test-generation.yml
name: AI Test Coverage Check
on:
  pull_request:
    types: [opened, synchronize]

jobs:
  check-test-coverage:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Identify changed files
        id: changed
        run: |
          echo "files=$(git diff --name-only origin/main...HEAD | grep '\.ts$' | head -20)" >> $GITHUB_OUTPUT

      - name: Check for missing tests
        run: |
          claude --print "Review these changed files and identify any that lack 
          corresponding test coverage: ${{ steps.changed.outputs.files }}
          
          For each file missing tests, output a JSON object with:
          - file: the source file path
          - test_file: where the test should go
          - scenarios: list of test cases needed"
```

## Structured Test Tracking with tests.json

Track test generation progress across sessions using a structured file:

```json
// tests.json
{
  "project": "e-commerce-api",
  "lastUpdated": "2025-01-15",
  "modules": [
    {
      "name": "auth",
      "path": "src/auth/",
      "tests": {
        "unit": { "status": "complete", "file": "src/auth/__tests__/auth.test.ts" },
        "integration": { "status": "in-progress", "file": "src/auth/__tests__/auth.integration.test.ts" },
        "e2e": { "status": "pending", "file": null }
      },
      "coverage": 78,
      "notes": "Need integration tests for OAuth2 flow"
    },
    {
      "name": "payments",
      "path": "src/payments/",
      "tests": {
        "unit": { "status": "complete", "file": "src/payments/__tests__/payments.test.ts" },
        "integration": { "status": "pending", "file": null },
        "e2e": { "status": "pending", "file": null }
      },
      "coverage": 52,
      "notes": "Stripe webhook handlers need test coverage"
    }
  ]
}
```

Prompt Claude Code to use this file:

```
> Read tests.json and work on the next "pending" module. 
> Write the tests, run them, and update the status to "complete" when done.
```

## Running Tests After Every Change

The verification loop is central to Claude Code's workflow. Configure it explicitly:

```markdown
<!-- CLAUDE.md -->

## Verification Protocol
After ANY code change:
1. Save all modified files
2. Run `npm test` for the affected module
3. If tests fail — fix the implementation, NOT the tests
4. Run full suite `npm test` before marking task complete
5. Run `npm run lint` to catch style issues

NEVER skip step 2. NEVER mark a task as done with failing tests.
```

Claude Code follows this loop naturally when instructed:

```
┌─────────────┐
│ Make change  │
└──────┬──────┘
       │
       ▼
┌─────────────┐     ┌──────────────┐
│  Run tests  │────▶│ Tests fail?  │
└─────────────┘     └──────┬───────┘
       ▲                   │ Yes
       │            ┌──────▼───────┐
       └────────────│  Fix code    │
                    └──────────────┘
       │ No
       ▼
┌─────────────┐
│  Task done   │
└─────────────┘
```

## Memory Tool for Test Context Persistence

Claude Code's memory (via `/memory` or the `CLAUDE.md` file) persists test context across sessions:

```
> /memory add "The payments module uses Stripe test mode API keys stored 
> in .env.test. Always use STRIPE_TEST_SECRET_KEY, never the live key. 
> Webhook tests require running stripe listen --forward-to localhost:3000/webhooks"
```

Useful test context to persist in memory:

| Context | Example |
|---------|---------|
| **Test database setup** | `"Run docker compose up -d postgres-test before integration tests"` |
| **Flaky test notes** | `"test/e2e/checkout.test.ts is flaky on CI — add retry(3)"` |
| **Environment variables** | `"Tests need DATABASE_URL=postgresql://localhost:5433/test"` |
| **Known limitations** | `"WebSocket tests don't work in CI — skip with @ci-skip tag"` |
| **Test data seeding** | `"Run npm run seed:test before running e2e tests"` |

## Complete CLAUDE.md Test Configuration Example

```markdown
<!-- CLAUDE.md -->

# Project: TaskFlow API

## Overview
A task management API built with Express + TypeScript + PostgreSQL.

## Test Stack
- **Unit tests:** Jest
- **Integration tests:** Jest + supertest  
- **E2E tests:** Playwright
- **Mocking:** MSW for external APIs, pg-mem for database

## Commands
| Action | Command |
|--------|---------|
| All unit tests | `npm test` |
| Single test file | `npm test -- path/to/file.test.ts` |
| Integration tests | `npm run test:integration` |
| E2E tests | `npm run test:e2e` |
| Coverage report | `npm run test:coverage` |
| Watch mode | `npm run test:watch` |

## Test File Locations
- Unit tests: alongside source (`src/services/user.test.ts`)
- Integration tests: `tests/integration/`
- E2E tests: `tests/e2e/`
- Fixtures: `tests/fixtures/`
- Factories: `tests/factories/`

## Rules
1. Run tests after EVERY change — no exceptions
2. New code requires tests — no untested code merges
3. Bug fixes require a regression test
4. Never modify tests to pass — fix the source code
5. Tests must be independent — no shared mutable state
6. Use factories (`tests/factories/`) for test data
7. External services are always mocked in unit tests
8. Integration tests use a dedicated test database

## Common Patterns
- See `tests/factories/user.factory.ts` for data factory example
- See `tests/integration/auth.test.ts` for authenticated API test pattern
- See `tests/e2e/pages/LoginPage.ts` for Page Object Model example
```
