# AI Agent Verification Loops

> How AI coding agents self-verify their changes through continuous test execution — and why skipping verification is the most dangerous anti-pattern.

---

## Overview

AI coding agents — GitHub Copilot Coding Agent, Claude Code, and others — are most effective when they run in a tight loop: write code, run tests, observe results, fix failures, repeat. This verification loop is what separates reliable AI-assisted development from reckless code generation. Without it, you're merging unverified changes.

## The Verification Loop

Every AI agent change should follow this cycle:

```
                    ┌─────────────────┐
                    │                 │
                    ▼                 │
              ┌───────────┐          │
              │ Write or  │          │
              │ modify    │          │
              │ code      │          │
              └─────┬─────┘          │
                    │                │
                    ▼                │
              ┌───────────┐          │
              │ Run tests │          │
              └─────┬─────┘          │
                    │                │
              ┌─────▼─────┐    ┌────┴─────┐
              │  Tests    │    │  Tests   │
              │  pass? ───┼───▶│  fail    │
              │  YES      │ NO │          │
              └─────┬─────┘    └────┬─────┘
                    │               │
                    ▼               │
              ┌───────────┐   ┌────▼─────┐
              │ Run lint  │   │ Analyze  │
              │ + type    │   │ failures │
              │ checks    │   └────┬─────┘
              └─────┬─────┘        │
                    │         ┌────▼─────┐
                    ▼         │ Fix the  │
              ┌───────────┐   │ CODE     │
              │ All green │   │ (not the │
              │ → Commit  │   │  tests)  │
              └───────────┘   └────┬─────┘
                                   │
                                   └──────▶ (back to top)
```

### Key Principles

1. **Every code change triggers a test run** — no exceptions
2. **Failures are fixed in the source code** — never in the tests
3. **The loop repeats until green** — partial passes aren't acceptable
4. **The final state is always verified** — run the full suite, not just the failing test

## Why Agents Should ALWAYS Run Tests

AI agents make mistakes. They hallucinate APIs, introduce subtle type errors, and miss edge cases. The test suite is their safety net:

| Without Verification | With Verification |
|---------------------|-------------------|
| Agent generates plausible-looking code | Agent generates code and runs it |
| Syntax errors slip through | Syntax errors caught immediately |
| Wrong API usage goes unnoticed | Tests fail on incorrect calls |
| Edge cases break in production | Edge cases caught by test suite |
| Silent data corruption | Assertions catch incorrect values |
| Broken imports and references | Module resolution errors caught |

**The cost of not verifying is always higher than the cost of running tests.**

## Configuring Verification in GitHub Copilot

### copilot-instructions.md

```markdown
<!-- .github/copilot-instructions.md -->

## Verification Requirements

After EVERY code change, you MUST:

1. Run `npm test` to verify unit tests pass
2. Run `npm run typecheck` to verify type safety
3. Run `npm run lint` to verify code style

If any step fails:
- Read the error output carefully
- Fix the SOURCE CODE, not the tests
- Re-run all verification steps
- Do not proceed until all checks pass

### Test Commands by Area
| Path | Test Command |
|------|-------------|
| `src/api/**` | `npm test -- --testPathPattern=src/api` |
| `src/services/**` | `npm test -- --testPathPattern=src/services` |
| `src/utils/**` | `npm test -- --testPathPattern=src/utils` |
| `tests/e2e/**` | `npx playwright test` |

### Full Verification (before completing any task)
```bash
npm run typecheck && npm run lint && npm test && npm run test:integration
```
```

### copilot-setup-steps.yml

Ensure the verification environment is ready:

```yaml
# .github/copilot-setup-steps.yml
steps:
  - uses: actions/checkout@v4
  - uses: actions/setup-node@v4
    with:
      node-version: '20'
      cache: 'npm'
  - run: npm ci
  - run: npx playwright install --with-deps chromium
  # Verify the test suite itself runs clean
  - run: npm test
```

## Configuring Verification in Claude Code

### CLAUDE.md

```markdown
<!-- CLAUDE.md -->

## Mandatory Verification Protocol

### After ANY Code Modification
1. Save all files
2. Run: `npm test` (unit tests for the changed module)
3. If tests fail → fix the implementation, run tests again
4. When unit tests pass → run: `npm run test:integration`
5. When all pass → run: `npm run lint && npm run typecheck`
6. Only mark task complete when ALL checks pass

### Critical Rules
- NEVER skip test execution to save time
- NEVER modify a test to make it pass
- NEVER mark a task complete with failing tests
- If you cannot fix a test failure after 3 attempts, STOP and explain the issue

### Quick Verification
For small changes: `npm test -- --bail --testPathPattern=<changed-file>`
For full verification: `npm run verify` (runs typecheck + lint + test + test:integration)
```

### Using Claude Code's Agentic Loop

Claude Code naturally operates in a verification loop. Reinforce it:

```
> Implement the user search feature. After each change:
> 1. Run the related tests
> 2. Fix any failures
> 3. Only move to the next step when tests pass
> Show me the test output at each step.
```

## Never Let the Agent Skip the Test Step

Agents sometimes try to skip verification when they're "confident" in their changes. This is always wrong.

### Warning Signs an Agent Is Skipping Verification

- "This change is straightforward, so tests should pass"
- "I've made the fix — let me move on to the next task"
- "The change is a simple rename, no need to test"
- "Tests are already passing from the previous change"

### Preventing Verification Skipping

In your instructions, be explicit:

```markdown
## Non-Negotiable Rules
- You MUST run tests after EVERY change, no matter how small
- "Simple" changes are the most likely to have unexpected side effects
- A rename can break imports, a type change can break serialization
- There is NO change small enough to skip verification
```

## ⚠️ Anti-Pattern: Agent Modifies Tests to Pass

The most dangerous anti-pattern occurs when an agent encounters a failing test and "fixes" it by changing the test assertions instead of the source code:

### The Problem

```typescript
// Original test (correct specification)
it('should return 404 for missing user', async () => {
  const response = await request(app).get('/users/nonexistent');
  expect(response.status).toBe(404);
  expect(response.body.error).toBe('User not found');
});

// ❌ Agent "fixes" the test instead of the code
it('should return 404 for missing user', async () => {
  const response = await request(app).get('/users/nonexistent');
  expect(response.status).toBe(500);  // Changed from 404 to 500!
  expect(response.body.error).toBe('Internal server error');  // Changed!
});
```

The test now passes, but the behavior is wrong. The specification was correct — the implementation is broken.

### How to Prevent This

**1. Explicit Instructions**

```markdown
## Test Modification Rules
- NEVER change test assertions to match incorrect behavior
- NEVER change expected values in tests to make them pass
- NEVER delete failing tests
- If a test expectation seems wrong, STOP and ask — do not change it
- The ONLY acceptable test modifications are:
  - Adding NEW tests
  - Updating test setup/teardown code
  - Fixing genuine test bugs (wrong mock setup, missing await)
```

**2. Code Review Checks**

When reviewing AI-generated PRs, always check:

```bash
# See what changed in test files
git diff main -- '**/*.test.*' '**/*.spec.*'
```

Look for changed `expect` values, removed assertions, or weakened conditions.

**3. Snapshot Tests as Guardrails**

Snapshot tests resist modification because changing them requires explicit acknowledgment:

```typescript
it('should format user response correctly', () => {
  const result = formatUserResponse(mockUser);
  expect(result).toMatchSnapshot();
  // Agent can't silently change the expected format
});
```

## Verification Across the Development Lifecycle

```
Feature Development:
  Write code → Run tests → Fix → Commit (per-file verification)

Pull Request:
  All unit tests → Integration tests → E2E tests → Coverage check

Pre-Merge:
  Full test suite → Lint → Type check → Security scan

Post-Deploy:
  Smoke tests → Health checks → Canary metrics
```

## Verification Checklist

Use this checklist for every AI agent task:

- [ ] Agent ran tests after every code change
- [ ] No existing tests were modified to pass
- [ ] No tests were deleted or skipped
- [ ] Full test suite passes (not just related tests)
- [ ] Lint and type checks pass
- [ ] New code has corresponding new tests
- [ ] Edge cases are covered
- [ ] Test output was reviewed (not just pass/fail)
