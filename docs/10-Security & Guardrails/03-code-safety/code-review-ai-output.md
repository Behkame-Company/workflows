# Code Review Practices for AI-Generated Code

> AI code looks syntactically correct and reads well — which makes it harder to review. Focus your attention on boundaries, trust transitions, and error paths.

---

## Why AI Code Needs Different Review Practices

Code written by AI is different from code written by humans in ways that affect how you should review it:

| Characteristic | Human-Written Code | AI-Generated Code |
|---------------|-------------------|-------------------|
| Syntax errors | Common | Rare |
| Style consistency | Varies | Usually consistent |
| Logic correctness | Usually right for known patterns | Correct on surface, may miss edge cases |
| Security awareness | Depends on developer | Inconsistent — may omit critical protections |
| Context awareness | Understands the codebase | Limited to what's in the context window |
| Novel bugs | Human-pattern bugs | AI-pattern bugs (different failure modes) |

The danger: AI code **passes the glance test**. It looks right, compiles, and may even pass basic tests. Reviewers need to slow down and probe deeper.

---

## The 80/20 of AI Code Review

Focus 80% of your review effort on three areas that catch 80% of bugs:

### 1. Boundaries — Where Data Enters and Exits

Every place where data crosses a trust boundary is a potential vulnerability:

```
User input → API handler    (validate and sanitize)
API handler → Database      (parameterize queries)
Database → API response     (filter sensitive fields)
API response → Browser      (escape for context)
Service → External API      (validate response)
File system → Application   (validate contents)
```

**Review questions at boundaries:**
- Is input validated with a schema before processing?
- Are queries parameterized?
- Is the response filtered to exclude internal fields?
- Are external API responses validated before use?

### 2. Trust Transitions — Where Privileges Change

Anywhere the code makes decisions about who can do what:

```python
# 🔍 Review carefully — trust transition
if user.role == "admin":
    delete_all_records()

# 🔍 Review carefully — trust transition
@app.route('/api/users/<user_id>/settings')
def update_settings(user_id):
    # Does this verify the authenticated user owns this user_id?
    settings = request.json
    db.update_user_settings(user_id, settings)
```

**Review questions at trust transitions:**
- Is authentication checked before authorization?
- Can a user manipulate IDs to access others' resources?
- Are role checks performed on the server (not just the client)?
- Is there a default-deny policy?

### 3. Error Paths — What Happens When Things Go Wrong

AI often generates happy-path code with minimal error handling:

```javascript
// 🔍 Review the catch block carefully
try {
  const user = await fetchUser(id);
  const orders = await fetchOrders(user.id);
  return formatResponse(user, orders);
} catch (error) {
  // AI often does one of these:
  // 1. Returns the full error (leaks info)
  // 2. Silently swallows it (hides bugs)
  // 3. Logs but doesn't handle (inconsistent state)
}
```

**Review questions on error paths:**
- Does the catch block leak internal details to the client?
- Is the error logged with enough context for debugging?
- Does failure leave the system in a consistent state?
- Are partial operations rolled back (transactions)?

---

## Review Checklist for AI-Generated Code

Use this checklist for every AI-generated PR:

### Input/Output Safety

- [ ] All external input is validated with a schema (zod, pydantic, etc.)
- [ ] Input validation happens at the boundary, not deep inside business logic
- [ ] No direct use of `req.body`, `req.params`, `req.query` without validation
- [ ] File uploads are validated for type, size, and content
- [ ] URL/redirect targets are validated against an allowlist

### Database Safety

- [ ] All queries use parameterized placeholders (no string interpolation)
- [ ] ORM queries don't pass raw user input to `.where()` or `.raw()`
- [ ] Bulk operations have reasonable limits
- [ ] Read-then-write patterns use transactions or locks

### Authentication & Authorization

- [ ] Authentication is checked via middleware, not inline in handlers
- [ ] Authorization verifies the user can access the *specific* resource
- [ ] No privilege escalation via user-supplied roles or permissions
- [ ] Rate limiting on sensitive endpoints (login, password reset, signup)
- [ ] Session tokens are cryptographically random and sufficiently long

### Secrets & Configuration

- [ ] No hardcoded API keys, passwords, tokens, or connection strings
- [ ] Secrets loaded from environment variables or secret manager
- [ ] No secrets logged in debug or error output
- [ ] `.env` files are in `.gitignore`

### Error Handling

- [ ] Error responses don't expose stack traces, file paths, or SQL queries
- [ ] Errors are logged server-side with sufficient context
- [ ] Catch blocks handle errors meaningfully (not empty or re-throwing raw)
- [ ] Failed operations don't leave data in inconsistent state

### Concurrency

- [ ] Shared resources are accessed within transactions or with locks
- [ ] No check-then-act patterns without synchronization
- [ ] Queue/worker operations are idempotent

### Dependencies

- [ ] No new unfamiliar dependencies added without justification
- [ ] Dependencies are pinned to specific versions
- [ ] No dependencies with known vulnerabilities (`npm audit`, `pip audit`)

---

## Red Flags in AI-Generated Code

Watch for these patterns that frequently indicate AI-generated problems:

### 1. Overly Complex Solutions

AI sometimes generates elaborate solutions when a simple one exists:

```python
# 🚩 Red flag — AI generated a complex regex for email validation
import re
email_pattern = re.compile(
    r'^[a-zA-Z0-9.!#$%&\'*+/=?^_`{|}~-]+@[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}'
    r'[a-zA-Z0-9])?(?:\.[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?)*$'
)

# ✅ Use a validation library instead
from pydantic import EmailStr
```

**Why this matters:** Complex code is harder to review and more likely to have edge cases.

### 2. Unfamiliar Dependencies

```json
// 🚩 Red flag — AI introduced a dependency you haven't heard of
{
  "dependencies": {
    "super-easy-auth": "^1.0.3"
  }
}
```

**Check:** Is this package maintained? How many downloads? When was the last update? Does it have known CVEs?

### 3. Magic Numbers and Unexplained Constants

```python
# 🚩 Red flag — where do these numbers come from?
def calculate_risk_score(user):
    base = 42
    if user.age < 25:
        return base * 1.7
    return base * 0.3
```

### 4. Try-Catch-Ignore Patterns

```javascript
// 🚩 Red flag — silently swallowing errors
try {
  await processPayment(order);
} catch (e) {
  // AI left this empty or with just a console.log
  console.log(e);
}
```

### 5. Hardcoded Configuration

```python
# 🚩 Red flag — hardcoded values that should be configurable
MAX_UPLOAD_SIZE = 10485760  # What if this needs to change?
API_URL = "https://api.production.example.com"  # Works locally?
TIMEOUT = 30  # Seconds? Milliseconds?
```

### 6. Missing Null/Undefined Checks

```typescript
// 🚩 Red flag — AI assumes the user always exists
async function getUserProfile(userId: string) {
  const user = await db.findUser(userId);
  return {
    name: user.name,        // What if user is null?
    email: user.email,
    avatar: user.profile.avatar  // What if profile is null?
  };
}

// ✅ With proper null checks
async function getUserProfile(userId: string) {
  const user = await db.findUser(userId);
  if (!user) {
    throw new NotFoundError(`User ${userId} not found`);
  }
  return {
    name: user.name,
    email: user.email,
    avatar: user.profile?.avatar ?? null
  };
}
```

### 7. Copy-Paste Patterns Across Different Contexts

AI may generate similar-looking code for different endpoints but miss context-specific requirements:

```javascript
// 🚩 Red flag — identical error handling copy-pasted across endpoints
// Public endpoint — this is fine
app.get('/api/public/posts', async (req, res) => { /* ... */ });

// Admin endpoint — should have auth middleware but doesn't
app.get('/api/admin/users', async (req, res) => { /* ... same pattern as above */ });
```

---

## Review Workflow for AI-Generated PRs

```
1. Read the PR description and intent
   └── Does the AI's interpretation match the requirement?

2. Check the file list
   └── Are there unexpected file changes?
   └── Any new dependencies?

3. Review boundaries first
   └── Input validation at API handlers
   └── Output filtering in responses
   └── Database query construction

4. Check trust transitions
   └── Auth/authz middleware on protected routes
   └── Resource ownership verification

5. Examine error paths
   └── Catch blocks that leak info
   └── Empty catch blocks
   └── Missing transaction rollbacks

6. Verify test coverage
   └── Are edge cases tested?
   └── Are error cases tested?
   └── Do tests actually assert meaningful behavior?

7. Run automated checks
   └── SAST scan results clean?
   └── All existing tests pass?
   └── No new security advisories?
```

---

## AI Code Review Automation Helpers

Augment human review with automated checks specifically designed for AI PRs:

```yaml
# .github/workflows/ai-pr-review-helpers.yml
name: AI PR Review Helpers

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  review-helpers:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Check for common AI patterns
        run: |
          echo "## AI Code Review Helper" >> $GITHUB_STEP_SUMMARY

          # Check for hardcoded secrets patterns
          if git diff origin/main...HEAD | grep -iE '(password|secret|api_key|token)\s*=\s*"[^"]{8,}"'; then
            echo "⚠️ Possible hardcoded secrets detected" >> $GITHUB_STEP_SUMMARY
          fi

          # Check for innerHTML usage
          if git diff origin/main...HEAD | grep -E 'innerHTML|document\.write|v-html'; then
            echo "⚠️ innerHTML/document.write usage detected — verify XSS safety" >> $GITHUB_STEP_SUMMARY
          fi

          # Check for string-interpolated SQL
          if git diff origin/main...HEAD | grep -E 'f".*SELECT|f".*INSERT|f".*UPDATE|f".*DELETE'; then
            echo "🚨 SQL string interpolation detected — use parameterized queries" >> $GITHUB_STEP_SUMMARY
          fi

          # Check for os.system
          if git diff origin/main...HEAD | grep -E 'os\.system\(|shell=True'; then
            echo "🚨 os.system or shell=True detected — use subprocess with arg list" >> $GITHUB_STEP_SUMMARY
          fi

          # Check for empty catch blocks
          if git diff origin/main...HEAD | grep -A1 'catch' | grep -E '^\+\s*\}'; then
            echo "⚠️ Possible empty catch block detected" >> $GITHUB_STEP_SUMMARY
          fi
```

---

## Tips

- ✅ Review AI code with a **security-first mindset** — assume vulnerabilities until proven otherwise
- ✅ Focus on boundaries, trust transitions, and error paths — this is where AI fails most
- ✅ Use the checklist above as a template — customize it for your project
- ✅ Run SAST before human review to eliminate obvious issues and focus reviewer attention
- ✅ Question every new dependency the AI introduces — verify it's real, maintained, and needed
- ❌ Don't trust AI code just because it looks clean and compiles
- ❌ Don't skip review because "the AI was given good instructions"
- ❌ Don't review AI code the same way you review human code — the failure modes are different
