# Anti-Pattern: Skipping Verification

> When developers trust AI output without running tests — the "it looks right" trap that ships plausible-looking bugs to production.

---

## The Problem

AI-generated code **looks** correct more often than it **is** correct. This creates a dangerous confidence gap:

```
Human-written code:                AI-generated code:
───────────────────                ──────────────────
Developer knows where              Code reads well
they cut corners                   Formatting is perfect
                                   Variable names are clear
"I should test this,               "This looks right,
 it's tricky"                       I'll just ship it"
```

The better AI code looks, the less likely developers are to verify it. This is the opposite of safe behavior.

---

## The "It Looks Right" Trap

### How It Happens

```markdown
1. Developer asks AI to implement a feature
2. AI generates clean, well-formatted code
3. Developer reads through it: "Looks reasonable"
4. Developer commits and pushes
5. No tests run. No verification.
6. Bug discovered in production 3 days later.
```

### Why AI Code Looks More Correct Than It Is

| Property | AI Code | Actual Correctness |
|----------|---------|-------------------|
| **Formatting** | Perfect indentation, naming | Irrelevant to correctness |
| **Structure** | Follows common patterns | Pattern may not fit the specific case |
| **Readability** | Clear, well-commented | Comments may describe wrong behavior |
| **Confidence** | Presents code definitively | No uncertainty expressed |
| **Completeness** | Appears complete at a glance | Often missing edge cases |

### Real Example

```javascript
// AI generates this — looks perfect at a glance
async function transferFunds(fromId, toId, amount) {
  const fromAccount = await Account.findById(fromId);
  const toAccount = await Account.findById(toId);

  if (fromAccount.balance < amount) {
    throw new Error('Insufficient funds');
  }

  fromAccount.balance -= amount;
  toAccount.balance += amount;

  await fromAccount.save();
  await toAccount.save();  // BUG: not in a transaction!
  // If this save fails, money is deducted but not deposited
}

// Looks correct ✅ (to a quick review)
// Is correct  ❌ (race condition + no transaction)
```

---

## The Statistics

AI-generated code contains subtle bugs in a significant portion of non-trivial tasks:

```
Typical AI Code Accuracy (non-trivial tasks):
──────────────────────────────────────────────
Trivial tasks (formatting, boilerplate):   ~95% correct
Simple functions (CRUD, utils):            ~80-90% correct
Complex logic (state machines, concurrency): ~60-80% correct
Security-sensitive code:                    ~50-70% correct

The gap between "looks right" and "is right"
grows with complexity.
```

---

## Common Bugs in Unverified AI Code

### Off-By-One Errors

```javascript
// AI-generated: processes items
for (let i = 0; i <= items.length; i++) {  // <= should be <
  process(items[i]);  // undefined on last iteration
}
```

### Missing Error Handling

```python
# AI-generated: reads config file
def load_config(path):
    with open(path) as f:    # No try/except — crashes on missing file
        return json.load(f)  # No handling for malformed JSON
```

### Incorrect Async Handling

```javascript
// AI-generated: fetches data
async function fetchAll(urls) {
  const results = [];
  urls.forEach(async (url) => {     // forEach doesn't await!
    const data = await fetch(url);
    results.push(await data.json());
  });
  return results;  // Returns empty array — forEach isn't awaited
}
```

### Security Vulnerabilities

```python
# AI-generated: user search
def search_users(query):
    sql = f"SELECT * FROM users WHERE name LIKE '%{query}%'"  # SQL injection!
    return db.execute(sql)
```

---

## Prevention Strategies

### Strategy 1: Mandatory Test Runs in CI

```yaml
# .github/workflows/ci.yml
name: Required Checks
on: [pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm test -- --ci --coverage
      - name: Enforce minimum coverage
        run: |
          COVERAGE=$(cat coverage/coverage-summary.json | jq '.total.lines.pct')
          if (( $(echo "$COVERAGE < 80" | bc -l) )); then
            echo "Coverage $COVERAGE% is below 80% threshold"
            exit 1
          fi

# Branch protection: require this check to pass before merge
```

### Strategy 2: Agent Verification Loops

Configure AI agents to verify their own output:

```markdown
# CLAUDE.md
## Verification Protocol
After writing ANY code:
1. Run the test suite: `npm test`
2. If tests fail, fix the code (not the tests)
3. If no tests exist for the new code, write tests FIRST
4. Never mark a task as complete without green tests
5. Report test results in your response
```

```markdown
# copilot-instructions.md
## Self-Verification
- Always run tests after generating code
- Include test output in your response
- If you can't run tests, explicitly warn that the code is UNVERIFIED
```

### Strategy 3: PR Gates

```yaml
# Branch protection rules (GitHub)
# Settings → Branches → Branch protection rules

Required status checks:
  - ci/test          # Tests must pass
  - ci/coverage      # Coverage must not decrease
  - ci/lint          # Linting must pass

Require reviews: 1
# Reviewer should verify tests were actually run, not just skipped
```

### Strategy 4: The Verification Checklist

```markdown
## Before Merging AI-Generated Code
- [ ] Tests exist for the new code
- [ ] Tests were run locally (paste output)
- [ ] Tests pass in CI (link to green build)
- [ ] Edge cases are tested (not just happy path)
- [ ] Error handling is tested
- [ ] I can explain what the code does (didn't just accept it)
- [ ] I checked for common AI bugs (off-by-one, async, security)
```

---

## Culture Shift: Never Trust, Always Verify

### The Mental Model

```
Human code:  "I wrote it, I might have bugs"     → Test it
AI code:     "AI wrote it, it looks perfect"      → DEFINITELY test it

The more confident you feel about AI code,
the more important it is to verify.
```

### Team Practices

```markdown
## Team Verification Standards

1. **No "AI wrote it" as justification**
   AI-generated code is held to the same standard as human code.

2. **Test output required in PRs**
   Every PR must include evidence that tests passed.

3. **"Did you run it?" is a valid review question**
   Reviewers should ask if the code was actually executed.

4. **Verification time is not wasted time**
   Running tests on AI code takes minutes. Debugging production bugs takes hours.

5. **Flag unverified code explicitly**
   If tests can't be run for some reason, mark the PR as "UNVERIFIED"
   and document why.
```

---

## Cost of Skipping Verification

```
Time to run tests:           2 minutes
Time to review AI code:      5 minutes
Time to debug production:    2-8 hours
Time for incident response:  4-24 hours
Time to rebuild trust:       Weeks

Verification is always cheaper than production bugs.
```

---

## Configuring AI to Self-Verify

```markdown
# CLAUDE.md
## Mandatory Verification
- ALWAYS run tests after code changes: `npm test`
- NEVER say "this should work" — run it and prove it
- Include test output in every response that includes code
- If tests don't exist, say so explicitly and offer to write them
- Consider your code UNVERIFIED until tests pass
```

```markdown
# copilot-instructions.md
## Code Verification
- Run the test suite after every code change
- Report: "X tests passed, Y failed"
- If no tests exist for changed code, write tests first
- Never assume code is correct — verify with tests
```

---

## Quick Reference: Verification Levels

| Level | Action | When |
|-------|--------|------|
| **Minimum** | Run existing tests | Every code change |
| **Standard** | Run tests + check coverage | Feature PRs |
| **Thorough** | Tests + coverage + mutation testing | Critical paths |
| **Maximum** | All above + manual smoke test | Security, payments, auth |
