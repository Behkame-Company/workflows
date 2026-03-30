# Templates

> Ready-to-use configuration templates for AI code review and CI/CD integration.

---

## Template 1: GitHub Actions — Full CI/CD Pipeline

```yaml
# .github/workflows/ci.yml
name: CI Pipeline
on:
  pull_request:
    branches: [main, develop]
  push:
    branches: [main]

concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'
      - run: npm ci
      - run: npm run lint
      - run: npm run format:check

  test:
    runs-on: ubuntu-latest
    needs: lint
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'
      - run: npm ci
      - run: npm test -- --coverage
      - name: Coverage gate
        run: |
          COVERAGE=$(npx coverage-summary --json | jq '.total.lines.pct')
          if (( $(echo "$COVERAGE < 80" | bc -l) )); then
            echo "Coverage $COVERAGE% is below 80% threshold"
            exit 1
          fi

  security:
    runs-on: ubuntu-latest
    needs: lint
    permissions:
      security-events: write
    steps:
      - uses: actions/checkout@v4
      - name: Initialize CodeQL
        uses: github/codeql-action/init@v3
        with:
          languages: javascript-typescript
      - name: Run CodeQL Analysis
        uses: github/codeql-action/analyze@v3

  build:
    runs-on: ubuntu-latest
    needs: [test, security]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'
      - run: npm ci
      - run: npm run build
      - uses: actions/upload-artifact@v4
        with:
          name: build
          path: dist/
```

---

## Template 2: copilot-instructions.md for Code Review

```markdown
# GitHub Copilot Instructions

## Code Review Rules

### Block (Critical)
- Hardcoded secrets, API keys, or passwords
- SQL injection vulnerabilities (non-parameterized queries)
- Missing authentication/authorization checks
- Direct use of eval() or similar dangerous functions

### Require Fix (High)
- Missing error handling on async operations
- Missing input validation on API endpoints
- Public functions without JSDoc documentation
- Functions exceeding 50 lines
- Missing unit tests for new features

### Warn (Medium)
- Complex conditionals (cyclomatic complexity >10)
- Magic numbers without named constants
- TODO comments without issue references
- Deeply nested callbacks (>3 levels)

### Ignore
- Import order (handled by ESLint)
- Formatting (handled by Prettier)
- CSS class naming conventions
- Test file structure
```

---

## Template 3: CodeRabbit Configuration

```yaml
# .coderabbit.yaml
language: en-US
reviews:
  profile: "chill"
  request_changes_workflow: false
  high_level_summary: true
  poem: false
  review_status: true
  collapse_walkthrough: false
  auto_review:
    enabled: true
    drafts: false
  path_instructions:
    - path: "src/api/**"
      instructions: "Check for input validation, error handling, and auth"
    - path: "src/db/**"
      instructions: "Verify parameterized queries and migration safety"
    - path: "**/*.test.*"
      instructions: "Check for meaningful assertions, not just snapshots"
  path_filters:
    - "!dist/**"
    - "!**/*.lock"
    - "!**/*.generated.*"
  tools:
    eslint:
      enabled: true
    biome:
      enabled: false
chat:
  auto_reply: true
```

---

## Template 4: Branch Protection Rules

```json
{
  "required_status_checks": {
    "strict": true,
    "contexts": [
      "lint",
      "test", 
      "security",
      "build",
      "copilot-review"
    ]
  },
  "required_pull_request_reviews": {
    "required_approving_review_count": 1,
    "dismiss_stale_reviews": true,
    "require_code_owner_reviews": true
  },
  "enforce_admins": true,
  "restrictions": null
}
```

---

## Template 5: CODEOWNERS

```
# .github/CODEOWNERS

# Default owner
* @team-lead

# Security-sensitive
src/auth/**        @security-team @team-lead
src/payments/**    @security-team @team-lead
*.env*             @security-team

# Infrastructure
.github/**         @devops-team
docker-compose*    @devops-team
Dockerfile         @devops-team

# Database
src/db/**          @backend-team
migrations/**      @backend-team @team-lead
```

---

## Template 6: PR Template

```markdown
<!-- .github/pull_request_template.md -->
## Description
<!-- What does this PR do? -->

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation

## Testing
- [ ] Unit tests added/updated
- [ ] Integration tests added/updated
- [ ] Manual testing done

## Checklist
- [ ] Code follows project style guide
- [ ] Self-review completed
- [ ] Documentation updated
- [ ] No new warnings introduced
```
