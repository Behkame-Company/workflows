# GitHub Copilot Code Review

> How to set up and use GitHub Copilot as an automated PR reviewer — natively on GitHub, in VS Code, and in CI/CD.

---

## Overview

GitHub Copilot Code Review is a built-in AI reviewer that analyzes pull requests and provides inline suggestions. It combines LLM-powered semantic analysis with deterministic tools (CodeQL, ESLint) for high-signal findings.

---

## Setup Methods

### Method 1: GitHub.com Repository Rulesets (Recommended)

Auto-review every PR via repository settings:

1. Navigate to **Repository → Settings → Rules → Rulesets**
2. Click **New ruleset**
3. Select target branches (e.g., `main`, `develop`)
4. Enable **"Automatically request Copilot code review"**
5. Options:
   - Review on new PR creation
   - Review on new pushes to existing PR
   - Review draft PRs
6. Save and activate

This can be enforced organization-wide for Copilot Business/Enterprise plans.

### Method 2: Manual Request

In any PR, type `@copilot review` in a comment or use the reviewer dropdown to request Copilot as a reviewer.

### Method 3: VS Code / IDE

Use Copilot's in-editor review features for pre-PR feedback:
- Select code → "Copilot: Review Selection"
- Get inline suggestions before pushing

---

## Customizing Review Behavior

### Using copilot-instructions.md

Create `.github/copilot-instructions.md` to guide Copilot's review focus:

```markdown
## Code Review Instructions

When reviewing pull requests:
- Flag any SQL query that doesn't use parameterized statements
- Ensure all API endpoints have input validation
- Verify that new public functions have JSDoc comments
- Check that error handling follows our ErrorResponse pattern
- Ignore formatting issues (Prettier handles this)
- Focus on security, performance, and correctness
```

### Using Path-Specific Instructions

Create `src/database/.instructions.md`:
```markdown
When reviewing database code:
- All queries must use the query builder, not raw SQL
- Check for N+1 query patterns
- Verify indexes exist for new WHERE clauses
```

---

## What Copilot Review Catches

| Category | Examples |
|----------|---------|
| **Security** | SQL injection, XSS, hardcoded secrets, insecure crypto |
| **Bugs** | Null dereference, off-by-one, race conditions, unchecked errors |
| **Performance** | N+1 queries, unnecessary re-renders, memory leaks |
| **Style** | Naming violations, dead code, overly complex functions |
| **Testing** | Missing test cases for new code paths |
| **Documentation** | Missing or outdated API docs |

---

## Review Output

Copilot posts review comments with:
- **Issue description**: What's wrong and why
- **Severity indicator**: Critical, warning, or suggestion
- **Code suggestion**: A fixable suggestion that can be applied with one click
- **Context**: Why this matters (security risk, performance impact, etc.)

---

## Pricing & Plans

| Plan | Code Review Support |
|------|-------------------|
| **Copilot Free** | Limited requests |
| **Copilot Pro** | Full review with PRUs |
| **Copilot Business** | Organization-wide enforcement |
| **Copilot Enterprise** | Custom models, knowledge bases |

---

## Key Takeaways

1. **Rulesets for automation** — Set up once, reviews every PR automatically
2. **Custom instructions are critical** — Guide Copilot to your team's standards
3. **Combines AI + deterministic** — LLM reasoning plus CodeQL/linter findings
4. **One-click fixes** — Suggestions can be applied directly in the PR
5. **Organization-wide enforcement** — Business/Enterprise plans for teams

---

## Next Steps

- 🔗 [CodeRabbit](coderabbit.md) — Alternative AI reviewer
- 🔗 [Review Instructions](../08-best-practices/review-instructions.md) — Writing effective review rules
