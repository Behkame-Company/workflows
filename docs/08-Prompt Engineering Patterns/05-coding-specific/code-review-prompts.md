# Code Review Prompts

> Prompts that produce high-signal code reviews — not noise.

---

## The Problem with Default Reviews

```
❌ "Review this code"

Result: Comments on naming, spacing, comment style, import order,
variable naming — lots of noise, few real issues.
```

---

## The High-Signal Review Prompt

```
Review this pull request for issues that could cause problems 
in production. ONLY flag:

1. Bugs — logic errors, off-by-one, null references
2. Security — injection, auth bypass, data exposure
3. Data loss — unhandled errors that could corrupt or lose data
4. Performance — O(n²) or worse on collections that could be large
5. Breaking changes — API contract violations, removed fields

Do NOT comment on:
- Code style or formatting
- Naming preferences
- Import organization
- Comment style or presence
- Minor refactoring opportunities

For each issue, provide:
- Severity: critical / high / medium
- Line(s): where the issue is
- Issue: one-sentence description
- Fix: the corrected code

If no issues found, say "No significant issues found."
```

---

## Focused Review Prompts

### Security-Focused
```
Review this code as a security auditor. Check for:
- SQL/NoSQL injection
- Cross-site scripting (XSS)
- Authentication/authorization bypass
- Sensitive data in logs or error messages
- Insecure cryptography or random number generation
- Path traversal
- SSRF

Assume the input comes from untrusted users.
```

### Performance-Focused
```
Review this code for performance issues. Assume:
- The users table has 1M rows
- The API handles 1000 requests/second
- The function is called on every page load

Flag:
- N+1 query patterns
- Missing database indexes (based on WHERE/JOIN clauses)
- Unnecessary data fetching (selecting * when only id/name needed)
- Unbounded queries (no LIMIT)
- Operations that should be batched
```

### API Contract Review
```
Review this API change for backward compatibility.

Compare the old and new interfaces:
- Are any required fields added to requests? (breaking)
- Are any response fields removed or renamed? (breaking)
- Are any enum values removed? (breaking)
- Are any endpoints removed or paths changed? (breaking)

Flag only actual breaking changes. Additive changes are OK.
```

---

## Review Severity Guide

Embed this in your review prompts to calibrate severity:

```
Severity definitions:
- Critical: Must fix before merge. Production crash, security vulnerability,
  data corruption.
- High: Should fix before merge. Likely bug, significant performance issue,
  breaking change.
- Medium: Should fix but not blocking. Edge case bug, minor performance concern.
- Low: Consider fixing. Maintainability concern, unclear code.
  (Only mention if no higher-severity issues found.)
```

---

## Team Review Template

```markdown
# copilot-instructions.md (code review section)

When reviewing code:
- Focus on bugs, security, and performance — not style
- Our style is enforced by ESLint and Prettier; ignore formatting
- Flag any `any` type usage as medium severity
- Flag missing error handling on async operations as high severity
- Flag unvalidated user input on API boundaries as critical
```
