# Code Quality Standards for AI Output

> Quality gates specific to AI-generated code. AI code often looks cleaner than human-written code but may contain subtle logic errors. These standards ensure AI output meets the same bar — or higher — as manually written code.

> For detailed quality gate implementation, CI/CD pipeline configuration, and AI-specific gate patterns, see [Section 06: Quality Gates](../../06-AI%20Code%20Review%20&%20CI-CD%20Integration/05-quality-gates/). This guide covers **team standards and review expectations**.

---

## The Quality Paradox

AI-generated code has a deceptive property: it often **looks** better than human code while being **less reliable**:

```
Human code:    May have style inconsistencies, but logic is intentional
AI code:       Style is perfect, but logic may contain subtle errors
```

This means traditional code review (which often focuses on style) is less effective for AI code. You need to **shift review focus to correctness**.

---

## Six Quality Standards

### Standard 1: All AI Code Must Pass Existing Linters and SAST

AI-generated code must clear the same automated checks as human code — no exceptions.

**Enforcement:**
```yaml
# CI/CD pipeline - same checks for all code
lint:
  runs-on: ubuntu-latest
  steps:
    - run: npm run lint          # ESLint, Prettier
    - run: npm run type-check    # TypeScript strict
    - run: npm run sast          # Semgrep, CodeQL, or similar
```

✅ AI code that passes all linters and SAST scans
❌ Disabling lint rules to accommodate AI output
❌ Skipping SAST because "AI doesn't generate vulnerabilities"

### Standard 2: AI Code Requires Same Test Coverage as Human Code

AI-generated code is not inherently trustworthy. Tests are the equalizer.

**Requirements:**
- New functions: unit tests required
- New endpoints: integration tests required
- Modified code: existing tests must still pass, new tests for new behavior
- Coverage threshold: same as team standard (e.g., 80% on new code)

**Enforcement:**
```yaml
# Coverage gate in CI
test:
  runs-on: ubuntu-latest
  steps:
    - run: npm test -- --coverage
    - run: npx coverage-check --threshold 80
```

✅ AI generates code AND tests that cover edge cases
❌ AI generates code, developer skips test generation
❌ AI-generated tests that only test happy path

### Standard 3: AI-Generated PRs Must Be Reviewed by a Human

No AI-generated code enters the main branch without human review — period.

**Requirements:**
- Minimum 1 human reviewer (2 for security-sensitive code)
- Reviewer must be able to explain what the code does
- AI disclosure in PR description (which tool, what scope)

**Enforcement:**
```yaml
# Branch protection rules
main:
  required_reviews: 1
  dismiss_stale_reviews: true
  require_code_owner_review: true  # For sensitive paths
```

✅ PR with clear AI disclosure and thoughtful human review
❌ Rubber-stamp approval of AI-generated PRs
❌ AI reviewing AI-generated code with no human in the loop

### Standard 4: AI Code Must Follow Team Style Guide

AI tools follow instruction files, but they're not perfect. Output must conform.

**Requirements:**
- Naming conventions match team standards
- File organization follows project structure
- Import ordering follows team preference
- Error handling uses team patterns
- Documentation follows team format

**Enforcement:**
- Automated: Linters and formatters catch most style issues
- Manual: Reviewers check for pattern consistency

✅ AI output that reads like the rest of the codebase
❌ AI output with different naming conventions, patterns, or organization

### Standard 5: No AI-Introduced Dependencies Without Approval

AI tools may suggest libraries or imports that aren't part of the project. All new dependencies require team approval.

**Requirements:**
- New npm/pip/etc. packages require team discussion
- AI-suggested imports must reference existing project code
- Hallucinated imports (non-existent packages) must be caught

**Enforcement:**
```yaml
# Dependency check in CI
deps:
  runs-on: ubuntu-latest
  steps:
    - run: npx lockfile-lint --type npm --allowed-hosts npm
    - run: npx audit-ci --moderate  # Check for vulnerable deps
```

**Review checklist for imports:**
```
[ ] All imports resolve to existing packages
[ ] No new dependencies added without discussion
[ ] No hallucinated packages (check npm registry)
[ ] Existing alternatives considered before adding new deps
```

✅ AI uses existing project dependencies
❌ AI introduces a new library without team awareness
❌ AI generates code importing a package that doesn't exist

### Standard 6: Security Scan Must Pass

AI-generated code goes through the same security scanning as human code, with additional focus on common AI mistakes.

**Requirements:**
- SAST scan passes (no new high/critical findings)
- Secret scanning passes (no leaked credentials)
- Dependency audit passes (no known vulnerabilities)
- Manual security review for auth/data access code

**Enforcement:**
```yaml
# Security gates in CI
security:
  runs-on: ubuntu-latest
  steps:
    - name: SAST
      run: npx semgrep --config auto --error
    - name: Secrets
      run: npx gitleaks detect --no-git
    - name: Dependencies
      run: npm audit --audit-level=high
```

✅ AI code that passes all security scans
❌ Suppressing security findings in AI-generated code
❌ Skipping security review because "AI handles security"

---

## What to Focus Review On

AI code review should emphasize these areas:

### Boundaries and Trust Transitions
```
Where does trusted data become untrusted?
- API inputs → validation needed
- Database results → null checking needed
- External API calls → error handling needed
- User uploads → sanitization needed
```

### Error Handling
```
What happens when things go wrong?
- Network timeout → handled?
- Invalid input → clear error message?
- Resource not found → appropriate response?
- Permission denied → no information leakage?
- Concurrent modification → race condition handled?
```

### Edge Cases
```
What are the unusual inputs?
- Empty string, null, undefined
- Very large numbers, negative numbers
- Unicode, special characters
- Empty arrays, single-element arrays
- Concurrent requests
```

### Security
```
Can this be exploited?
- Injection (SQL, NoSQL, command, LDAP)
- Authentication bypass
- Authorization escalation
- Data exposure
- Denial of service
```

---

## Quality Gate Configuration

### Combined CI Pipeline

```yaml
name: AI Code Quality Gates

on:
  pull_request:
    branches: [main, develop]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm run lint
      - run: npm run type-check

  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm test -- --coverage
      - name: Check coverage
        run: |
          npx coverage-check --threshold 80

  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm audit --audit-level=high
      - name: SAST scan
        uses: returntocorp/semgrep-action@v1
        with:
          config: auto
      - name: Secret scan
        uses: gitleaks/gitleaks-action@v2

  review-gate:
    runs-on: ubuntu-latest
    needs: [lint, test, security]
    steps:
      - name: All quality gates passed
        run: echo "Ready for human review"
```

---

## Measuring Quality Over Time

Track these metrics monthly:

| Metric | What it tells you | Target |
|---|---|---|
| AI-related bug rate | Quality of review process | Decreasing quarter over quarter |
| Test coverage on AI code | Thoroughness of testing | ≥ team standard |
| SAST findings in AI code | Security quality | Zero critical/high |
| PR revision rounds | First-submission quality | Decreasing over time |
| Time to review AI PRs | Review efficiency | Stable or decreasing |
