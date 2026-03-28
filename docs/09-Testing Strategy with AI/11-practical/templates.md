# Testing Templates

> Ready-to-use templates for AI-assisted testing — copy, customize, and integrate into your project.

---

## Template 1: Test Plan

Use this template at the start of a feature to plan your testing approach:

```markdown
# Test Plan: [Feature Name]

## Overview
- **Feature:** [Brief description]
- **Risk Level:** [Low / Medium / High / Critical]
- **Target Coverage:** [e.g., 85% line + 70% mutation score]

## Test Strategy
| Test Type | Count | Framework | Location |
|-----------|-------|-----------|----------|
| Unit      | ~[N]  | [Jest/Pytest] | tests/unit/ |
| Integration | ~[N] | [Jest/Pytest] | tests/integration/ |
| E2E       | ~[N]  | [Playwright/Cypress] | tests/e2e/ |

## Critical Paths to Test
1. [Happy path: user does X → system does Y]
2. [Error path: invalid input → error message]
3. [Edge case: boundary condition → expected behavior]

## Boundaries and Edge Cases
- [ ] Empty input
- [ ] Maximum input size
- [ ] Null/undefined values
- [ ] Concurrent access
- [ ] Network failure
- [ ] Unauthorized access

## Dependencies to Mock
- [ ] [External API] — mock with [approach]
- [ ] [Email service] — mock with [approach]

## Acceptance Criteria
- [ ] All tests pass
- [ ] Coverage target met
- [ ] No flaky tests
- [ ] Mutation score above threshold
```

---

## Template 2: tests.json Tracking

Machine-readable test tracking for AI sessions:

```json
{
  "meta": {
    "project": "my-project",
    "feature": "user-authentication",
    "lastUpdated": "2024-01-01T00:00:00Z",
    "lastSession": "Initial test plan",
    "summary": {
      "total": 0,
      "pass": 0,
      "fail": 0,
      "pending": 0
    }
  },
  "tests": [
    {
      "name": "Descriptive test name here",
      "file": "tests/feature.test.js",
      "status": "pending",
      "type": "unit",
      "priority": "high",
      "notes": "Any context for the AI"
    }
  ]
}
```

### Status Values

| Status | Meaning |
|--------|---------|
| `pending` | Planned but not yet written |
| `written` | Test code exists but hasn't been run |
| `pass` | Test passes |
| `fail` | Test fails (include `error` field) |
| `skip` | Intentionally skipped (include `reason` field) |
| `flaky` | Non-deterministic (include `flakeRate` field) |

---

## Template 3: copilot-instructions.md Test Section

Add this to your project's Copilot custom instructions:

```markdown
## Testing

### Commands
- Run all tests: `npm test`
- Run specific file: `npm test -- --testPathPattern="[pattern]"`
- Run with coverage: `npm test -- --coverage`
- Update snapshots: `npm test -- --updateSnapshot`

### Conventions
- Test files: `[name].test.ts` co-located with source files
- Test IDs: use `data-testid` attributes for E2E selectors
- Factories: use `tests/factories/` for test data generation
- Fixtures: use `tests/fixtures/` for static test data

### Rules
- Write tests BEFORE implementation (TDD)
- Never modify existing tests to make new code pass
- Run full test suite after every code change
- Minimum coverage: 80% lines, 70% branches
- No `test.skip` without a linked issue number
- Use descriptive test names: "does X when Y" format

### Mocking
- Mock external HTTP APIs with `msw` (Mock Service Worker)
- Mock email with test transport
- Use real test database (SQLite in-memory)
- Never mock more than 1 layer deep

### Test Tracking
- Read `tests.json` before starting any testing task
- Update `tests.json` after running tests
```

---

## Template 4: CLAUDE.md Test Section

Add this to your CLAUDE.md for Claude Code projects:

```markdown
## Testing

### Quick Reference
- Run tests: `npm test`
- Run + coverage: `npm test -- --coverage`
- Single file: `npm test -- path/to/test`
- Watch mode: `npm test -- --watch`

### Workflow
1. Before coding: check `tests.json` for current status
2. Write failing tests first (TDD)
3. Implement to make tests pass
4. Run full suite after every change
5. Update `tests.json` with results

### Mandatory Rules
- ALWAYS run tests after code changes
- NEVER modify existing tests to fix your code
- NEVER use `test.skip` without justification
- Include test output in your responses
- If no tests exist for changed code, write them first

### Test Structure
- Unit tests: `tests/unit/[module].test.ts`
- Integration: `tests/integration/[feature].test.ts`
- E2E: `tests/e2e/[flow].spec.ts`
- Fixtures: `tests/fixtures/`
- Factories: `tests/factories/`
```

---

## Template 5: .prompt.md for Test Generation

Create prompt files that AI tools can reference:

```markdown
---
name: Generate Tests
description: Generate comprehensive tests for a module
---

# Test Generation Prompt

Generate tests for the module at `{{filePath}}`.

## Requirements
- Use {{framework}} (e.g., Jest, Pytest, JUnit)
- Follow these test categories:

### Happy Path Tests
- Test the primary use case with valid input
- Test all public methods/functions
- Test return values AND side effects

### Edge Cases
- Empty/null/undefined inputs
- Boundary values (0, -1, MAX_INT)
- Very large inputs
- Special characters and Unicode
- Concurrent access (if applicable)

### Error Handling
- Invalid input types
- Missing required fields
- Network/IO failures (if applicable)
- Permission/authorization failures

## Style
- Descriptive test names: "[verb] [scenario] [expected result]"
- One assertion per test (prefer)
- Use factories for test data, not raw objects
- Group related tests with `describe` blocks

## Anti-Patterns to Avoid
- Do NOT copy implementation logic into assertions
- Do NOT use snapshots for complex components
- Do NOT mock internal modules
- Do NOT use `setTimeout` for async waiting
```

---

## Template 6: GitHub Actions Test Workflow

```yaml
# .github/workflows/test.yml
name: Test Suite

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [18.x, 20.x]

    steps:
      - uses: actions/checkout@v4

      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run linter
        run: npm run lint

      - name: Run tests with coverage
        run: npm test -- --ci --coverage --maxWorkers=2

      - name: Check coverage threshold
        run: |
          COVERAGE=$(node -e "
            const r = require('./coverage/coverage-summary.json');
            console.log(r.total.lines.pct);
          ")
          echo "Line coverage: $COVERAGE%"
          if (( $(echo "$COVERAGE < 80" | bc -l) )); then
            echo "::error::Coverage $COVERAGE% is below 80% threshold"
            exit 1
          fi

      - name: Upload coverage report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: coverage-report-${{ matrix.node-version }}
          path: coverage/

  e2e:
    runs-on: ubuntu-latest
    needs: test

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20.x
          cache: 'npm'

      - run: npm ci

      - name: Install Playwright browsers
        run: npx playwright install --with-deps

      - name: Run E2E tests
        run: npx playwright test

      - name: Upload E2E report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: playwright-report
          path: playwright-report/
```

---

## Template 7: Test Quality Review Checklist

Use during PR reviews:

```markdown
# Test Quality Review

## PR: [#number] [title]
## Reviewer: [name]
## Date: [date]

### Test Coverage
- [ ] New code has corresponding tests
- [ ] Coverage did not decrease (check CI report)
- [ ] Critical paths have integration tests

### Test Quality
- [ ] Tests verify behavior, not implementation
- [ ] Test names describe scenarios clearly
- [ ] Edge cases are covered
- [ ] Error handling is tested
- [ ] No tautological tests (tests that can't fail)

### AI-Specific Checks
- [ ] Tests were not auto-generated to match implementation
- [ ] No existing tests were modified to accommodate new code
- [ ] Mocking is appropriate (not over-mocked)
- [ ] No snapshot abuse (snapshots < 50 lines each)
- [ ] No timing-dependent assertions (setTimeout, sleep)

### Test Reliability
- [ ] Tests don't depend on execution order
- [ ] No shared mutable state between tests
- [ ] No hardcoded ports, paths, or timestamps
- [ ] Async operations properly awaited

### Verdict
- [ ] **APPROVE** — tests are thorough and well-written
- [ ] **REQUEST CHANGES** — issues noted above
- [ ] **NEEDS DISCUSSION** — testing approach needs team input
```

---

## Using Templates with AI

### Copilot

```markdown
# Reference the template in your prompt:
"Follow the test generation template in .prompts/generate-tests.prompt.md
 to create tests for src/services/payment.js"
```

### Claude Code

```markdown
# Reference in CLAUDE.md:
"For test generation, follow the patterns in .prompts/generate-tests.prompt.md"
```

### Any AI Tool

```markdown
# Copy the relevant template section into your prompt:
"Generate tests following this structure: [paste template]"
```

---

## Next Steps

- [Troubleshooting AI Testing](troubleshooting.md) — when things go wrong
- [Test Prompt Patterns](../09-best-practices/prompt-patterns.md) — effective prompts for each test type
- [Structured Test Tracking](../09-best-practices/structured-test-tracking.md) — the tests.json approach in depth
- [CI/CD Test Integration](../06-ai-qa-workflows/ci-cd-integration.md) — using the workflow template

---

*Part of the [Testing Strategy with AI](../README.md) series.*
