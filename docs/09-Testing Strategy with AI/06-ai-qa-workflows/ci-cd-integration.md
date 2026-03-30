# Test Integration in CI/CD

> Integrating AI-assisted testing into continuous integration and delivery pipelines for automated quality gates and consistent verification.

---

## Overview

AI-generated tests are only valuable if they run consistently. Integrating them into your CI/CD pipeline ensures every pull request is validated, every merge is tested, and regressions are caught before they reach production. This guide covers GitHub Actions workflows, PR gates, matrix testing, and reporting for AI-assisted test suites.

## GitHub Actions Workflow for AI-Generated Tests

A complete workflow that runs your test suite on every pull request:

```yaml
# .github/workflows/test.yml
name: Test Suite
on:
  pull_request:
    branches: [main, develop]
  push:
    branches: [main]

permissions:
  contents: read
  checks: write
  pull-requests: write

jobs:
  unit-tests:
    name: Unit Tests
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - run: npm ci

      - name: Run unit tests
        run: npm test -- --ci --coverage --reporters=default --reporters=jest-junit
        env:
          JEST_JUNIT_OUTPUT_DIR: ./reports
          JEST_JUNIT_OUTPUT_NAME: unit-tests.xml

      - name: Upload coverage
        uses: actions/upload-artifact@v4
        with:
          name: coverage-report
          path: coverage/

      - name: Test report
        uses: dorny/test-reporter@v1
        if: always()
        with:
          name: Unit Test Results
          path: reports/unit-tests.xml
          reporter: jest-junit

  integration-tests:
    name: Integration Tests
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_DB: testdb
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - run: npm ci

      - name: Run migrations
        run: npm run db:migrate
        env:
          DATABASE_URL: postgresql://test:test@localhost:5432/testdb

      - name: Run integration tests
        run: npm run test:integration -- --ci
        env:
          DATABASE_URL: postgresql://test:test@localhost:5432/testdb

  e2e-tests:
    name: E2E Tests
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - run: npm ci
      - run: npx playwright install --with-deps chromium

      - name: Build application
        run: npm run build

      - name: Run E2E tests
        run: npx playwright test
        env:
          BASE_URL: http://localhost:3000

      - name: Upload Playwright report
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: playwright-report
          path: playwright-report/
```

## Pre-Commit Hooks for Test Validation

Catch issues before they reach CI using Git hooks:

```json
// package.json
{
  "lint-staged": {
    "src/**/*.{ts,tsx}": [
      "eslint --fix",
      "jest --bail --findRelatedTests --passWithNoTests"
    ]
  }
}
```

```yaml
# .husky/pre-commit
npm run lint-staged
```

This runs only the tests related to changed files — fast feedback without running the full suite.

## PR Gates: All Tests Must Pass

Configure branch protection rules to enforce test passage:

```yaml
# Required status checks in branch protection:
# ✅ Unit Tests
# ✅ Integration Tests  
# ✅ E2E Tests
# ✅ Coverage threshold met
```

### Coverage Thresholds as Gates

```yaml
      - name: Check coverage threshold
        run: |
          COVERAGE=$(npx istanbul-cobertura-coverage --threshold 80 coverage/cobertura-coverage.xml 2>&1)
          if echo "$COVERAGE" | grep -q "FAIL"; then
            echo "❌ Coverage below 80% threshold"
            exit 1
          fi
          echo "✅ Coverage threshold met"
```

### Enforcing Test Existence for New Code

```yaml
      - name: Verify tests exist for changed files
        run: |
          CHANGED_SRC=$(git diff --name-only origin/main...HEAD | grep '^src/' | grep -v '\.test\.' | grep '\.ts$')
          MISSING_TESTS=""
          for file in $CHANGED_SRC; do
            TEST_FILE="${file%.ts}.test.ts"
            if [ ! -f "$TEST_FILE" ]; then
              MISSING_TESTS="$MISSING_TESTS\n  - $file (expected: $TEST_FILE)"
            fi
          done
          if [ -n "$MISSING_TESTS" ]; then
            echo "❌ Missing test files for:$MISSING_TESTS"
            exit 1
          fi
```

## AI Test Generation in PR Review Workflows

Trigger AI test generation as part of the PR review process:

```yaml
# .github/workflows/ai-test-review.yml
name: AI Test Review
on:
  pull_request:
    types: [opened, synchronize]

jobs:
  ai-test-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Identify untested changes
        id: coverage-check
        run: |
          # Run tests with coverage for changed files only
          CHANGED=$(git diff --name-only origin/main...HEAD | grep '\.ts$' | grep -v '\.test\.')
          echo "changed_files=$CHANGED" >> $GITHUB_OUTPUT
          npm test -- --coverage --changedSince=origin/main

      - name: Comment on PR with coverage gaps
        if: steps.coverage-check.outputs.changed_files != ''
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');
            const coverage = JSON.parse(fs.readFileSync('coverage/coverage-summary.json', 'utf8'));
            
            let comment = '## 🧪 Test Coverage Report\n\n';
            comment += '| File | Statements | Branches | Functions |\n';
            comment += '|------|-----------|----------|----------|\n';
            
            for (const [file, data] of Object.entries(coverage)) {
              if (file === 'total') continue;
              comment += `| ${file} | ${data.statements.pct}% | ${data.branches.pct}% | ${data.functions.pct}% |\n`;
            }
            
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: comment
            });
```

## Copilot Coding Agent Runs Tests Automatically

When using the Copilot Coding Agent, it automatically runs tests defined in your setup:

```
PR created by Copilot → setup steps run → agent makes changes → 
agent runs tests → fixes failures → pushes passing code
```

Configure the test commands the agent uses:

```markdown
<!-- .github/copilot-instructions.md -->
## Testing
After every code change, run:
1. `npm test` — unit tests
2. `npm run lint` — code style
3. `npm run typecheck` — type safety
```

```yaml
# .github/copilot-setup-steps.yml
steps:
  - uses: actions/checkout@v4
  - uses: actions/setup-node@v4
    with:
      node-version: '20'
  - run: npm ci
  - run: npx playwright install --with-deps
```

## Matrix Testing Across Environments

Test AI-generated code across multiple environments to catch compatibility issues:

```yaml
jobs:
  test-matrix:
    name: Test (${{ matrix.node-version }}, ${{ matrix.os }})
    runs-on: ${{ matrix.os }}
    strategy:
      fail-fast: false
      matrix:
        node-version: [18, 20, 22]
        os: [ubuntu-latest, windows-latest, macos-latest]
        exclude:
          # Skip expensive combinations
          - os: macos-latest
            node-version: 18
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm ci
      - run: npm test
```

⚠️ **AI-generated tests may unintentionally use platform-specific features.** Matrix testing catches path separators, line endings, and OS-specific API differences.

## Reporting and Visibility

### Test Result Summaries in PRs

```yaml
      - name: Create job summary
        if: always()
        run: |
          echo "## 🧪 Test Results" >> $GITHUB_STEP_SUMMARY
          echo "" >> $GITHUB_STEP_SUMMARY
          echo "| Suite | Status | Duration |" >> $GITHUB_STEP_SUMMARY
          echo "|-------|--------|----------|" >> $GITHUB_STEP_SUMMARY
          echo "| Unit Tests | ✅ Pass | 12s |" >> $GITHUB_STEP_SUMMARY
          echo "| Integration | ✅ Pass | 45s |" >> $GITHUB_STEP_SUMMARY
          echo "| E2E | ✅ Pass | 2m 30s |" >> $GITHUB_STEP_SUMMARY
```

### Trend Tracking

Store test results over time to detect quality drift:

```yaml
      - name: Record test metrics
        run: |
          echo "{
            \"date\": \"$(date -u +%Y-%m-%dT%H:%M:%SZ)\",
            \"commit\": \"${{ github.sha }}\",
            \"tests\": $(npm test -- --json 2>/dev/null | jq '.numTotalTests'),
            \"passed\": $(npm test -- --json 2>/dev/null | jq '.numPassedTests'),
            \"coverage\": $(cat coverage/coverage-summary.json | jq '.total.statements.pct')
          }" >> test-metrics.jsonl
```

## CI/CD Pipeline Architecture

```
┌─────────────┐     ┌──────────────┐     ┌──────────────┐
│   PR Opens   │────▶│  Lint + Type  │────▶│  Unit Tests  │
└─────────────┘     │   Check       │     │              │
                    └──────────────┘     └──────┬───────┘
                                                │ Pass
                    ┌──────────────┐     ┌──────▼───────┐
                    │  Coverage    │◀────│ Integration  │
                    │  Report      │     │ Tests        │
                    └──────┬───────┘     └──────────────┘
                           │
                    ┌──────▼───────┐     ┌──────────────┐
                    │  E2E Tests   │────▶│  PR Status   │
                    │  (matrix)    │     │  ✅ or ❌     │
                    └──────────────┘     └──────────────┘
```
