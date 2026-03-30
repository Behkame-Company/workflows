# Code Review Instructions

> Custom instructions for optimizing Copilot Code Review output.

---

## Overview

Copilot Code Review analyzes pull request diffs and suggests improvements. Custom instructions shape what it focuses on.

| Property | Value |
|----------|-------|
| **Reads** | copilot-instructions.md + matching .instructions.md |
| **Character limit** | ~4,000 characters processed |
| **Trigger** | Automatic on PR, or manual request |
| **Output** | Inline review comments on PR diff |

---

## Key Constraint: 4,000 Character Limit

Code Review processes approximately the first **4,000 characters** of custom instructions. Everything beyond is silently ignored.

**4,000 characters ≈ 600-700 words ≈ 1.5 pages**

This means:
- Your most important review rules must be in the first 4,000 chars
- If copilot-instructions.md is long, review-critical rules may be cut off
- Consider using `excludeAgent: "code-review"` on non-review instructions

---

## What to Include for Code Review

### High-Impact Review Rules

```markdown
## Code Review Focus Areas
- Security: SQL injection, XSS, credential exposure
- Type safety: no `any`, no unsafe type assertions
- Error handling: all async operations must have error handling
- Input validation: API endpoints must validate input
- Naming: descriptive function and variable names
- Tests: new logic must have accompanying tests
```

### Framework-Specific Rules

```markdown
## React Review Rules
- No direct DOM manipulation in components
- No state mutations (always create new references)
- useEffect cleanup functions for subscriptions
- Memoization (useMemo/useCallback) only when profiling shows need
```

### Security-Focused Rules

```markdown
## Security Review
- Flag any hardcoded secrets or API keys
- Verify parameterized queries (no string concatenation in SQL)
- Check for proper authentication on new endpoints
- Verify CORS configuration on API routes
- Flag any use of `dangerouslySetInnerHTML`
```

---

## Structuring for Code Review

### Optimized Layout (Under 4,000 Characters)

```markdown
# Code Review Instructions

## Critical (Always Flag)
- Hardcoded secrets or credentials
- SQL injection vulnerabilities
- Missing input validation on public endpoints
- Unhandled promise rejections
- Use of `any` type

## Important (Usually Flag)
- Missing error handling for async operations
- Missing tests for new logic
- Console.log statements left in code
- Unused imports or variables
- Magic numbers without constants

## Style (Flag if Consistent Pattern)
- Inconsistent naming conventions
- Missing JSDoc on exported functions
- Default exports (should be named)
- Direct DOM manipulation in React components
```

### Using Severity Levels

Group rules by severity so Code Review can prioritize:

```markdown
## 🔴 Block (Must fix before merge)
- Security vulnerabilities
- Missing error handling on external calls
- Breaking API changes without version bump

## 🟡 Warn (Should fix, PR can merge)
- Missing tests for new logic
- Type assertions without justification
- TODO comments without issue references

## 🔵 Info (Suggestions for improvement)
- Opportunities to simplify code
- Performance optimizations
- Documentation improvements
```

---

## Using excludeAgent for Code Review

Some instructions are irrelevant for code review. Use `excludeAgent` to prevent them from consuming the 4,000 character budget:

```yaml
---
applyTo: "src/components/**/*.tsx"
excludeAgent: "code-review"
---
# These instructions are for Chat and Coding Agent ONLY
# Code Review won't waste its budget on these

Detailed component creation workflow...
Step-by-step implementation guide...
(These are useful for generation but not review)
```

Conversely, create review-only instructions:

```yaml
---
applyTo: "src/app/api/**/*.ts"
---
# API Code Review Focus
When reviewing API routes:
- Verify input validation exists
- Check authentication middleware is applied
- Confirm proper HTTP status codes
- Verify error responses don't leak internals
```

---

## Review Instruction Anti-Patterns

### ❌ Too Generic

```markdown
Review code for best practices.
Ensure code quality is high.
```

**Problem**: Copilot doesn't know what "best practices" means to your team.

### ❌ Too Many Rules

```markdown
50 specific rules covering every aspect of code...
```

**Problem**: Exceeds 4,000 characters. Rules at the bottom are ignored.

### ❌ Contradicting Rules

```markdown
- Always prefer short functions (under 20 lines)
- Include comprehensive error handling in every function
```

**Problem**: Comprehensive error handling often pushes functions over 20 lines.

### ❌ Implementation Instructions in Review

```markdown
When creating new components, follow this 10-step process...
```

**Problem**: Code review looks at existing code, not creating new code. This wastes review context.

---

## Measuring Code Review Effectiveness

### Track Review Quality

| Metric | How to Measure |
|--------|---------------|
| True positives | Review caught a real issue |
| False positives | Review flagged something that's actually fine |
| False negatives | Review missed an issue found later |
| Usefulness | Developers find comments helpful |

### Iterate Based on Feedback

```
Month 1: "Review for security and type safety"
Feedback: "Too many false positives on type assertions"
Month 2: "Review for security. Type assertions are acceptable for [specific cases]"
Feedback: "Good, but missing API validation checks"
Month 3: "Review for security and API input validation..."
```

Adjust instructions based on what the team actually needs.

---

## Example: Production-Ready Code Review Instructions

```markdown
# Code Review Instructions

## Always Flag
- Secrets or credentials in code (API keys, passwords, tokens)
- SQL/NoSQL injection (string concatenation in queries)
- Missing authentication on API endpoints in `src/app/api/`
- Unhandled promise rejections or missing error boundaries
- `console.log` or debugging artifacts

## Type Safety
- Use of `any` type (suggest `unknown` + type guard instead)
- Unsafe type assertions (`as` without runtime check)
- Missing return types on exported functions

## API Routes (src/app/api/)
- Input must be validated with Zod schema
- Responses must use typed response helpers
- Error responses must not expose stack traces
- Authentication middleware applied to protected routes

## React Components (src/components/)
- No direct state mutations
- useEffect must have cleanup for subscriptions
- Event handlers must not be defined inline in JSX for complex components
- Prop interfaces must be exported

## Testing
- New logic files must have corresponding test files
- Test names should describe behavior: "should X when Y"
- Mock external dependencies, don't hit real services

## Do Not Flag
- Import ordering (handled by ESLint)
- Formatting (handled by Prettier)
- Line length (handled by config)
```

**Character count: ~1,200 — well under the 4,000 limit** ✅
