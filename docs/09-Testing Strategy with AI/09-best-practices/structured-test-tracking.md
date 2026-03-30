# Structured Test Tracking

> Tracking test status across sessions and agents using a machine-readable format — so AI tools always know which tests exist, which pass, and what's still pending.

---

## The Problem

AI sessions are stateless. When context resets:

- AI doesn't know which tests already exist
- AI doesn't know which tests are passing or failing
- AI may regenerate tests that already exist
- AI may skip tests that were planned but not yet written
- Multiple agents working in parallel can't coordinate

```
Session 1: "Write tests for auth module"
  → Writes 5 tests, 3 pass, 2 pending

  --- context reset ---

Session 2: "Continue testing"
  → "What tests? I don't see any context..."
  → Regenerates from scratch, duplicating work
```

---

## The tests.json Approach

A JSON file that serves as the single source of truth for test status:

### Basic Format

```json
{
  "tests": [
    {
      "name": "User can register with valid email",
      "file": "tests/auth/registration.test.js",
      "status": "pass",
      "line": 12
    },
    {
      "name": "Registration rejects duplicate email",
      "file": "tests/auth/registration.test.js",
      "status": "pass",
      "line": 24
    },
    {
      "name": "Login returns JWT for valid credentials",
      "file": "tests/auth/login.test.js",
      "status": "fail",
      "line": 8,
      "error": "Expected 200, received 401"
    },
    {
      "name": "Password reset sends email",
      "file": "tests/auth/password-reset.test.js",
      "status": "pending",
      "notes": "Needs email mock setup"
    }
  ]
}
```

### Extended Format

```json
{
  "meta": {
    "project": "my-app",
    "lastUpdated": "2024-03-15T10:30:00Z",
    "totalTests": 45,
    "passing": 38,
    "failing": 3,
    "pending": 4
  },
  "tests": [
    {
      "name": "User can register with valid email",
      "file": "tests/auth/registration.test.js",
      "status": "pass",
      "category": "auth",
      "type": "integration",
      "priority": "high",
      "addedBy": "session-001",
      "lastRun": "2024-03-15T10:28:00Z"
    }
  ]
}
```

---

## Benefits of Structured Tracking

| Benefit | How It Helps |
|---------|-------------|
| **Persistence** | Survives context resets — always available |
| **Shared state** | Multiple agents read the same file |
| **Machine-readable** | AI can parse, query, and update it automatically |
| **Auditability** | Track what was tested, when, and by whom |
| **Progress tracking** | Clear view of pending work |
| **Deduplication** | AI checks before generating duplicate tests |

---

## Using tests.json in Your Workflow

### Step 1: Initialize

```json
// tests.json — start of project
{
  "tests": []
}
```

### Step 2: Plan Tests

```json
{
  "tests": [
    { "name": "Creates user with valid data", "file": "tests/user.test.js", "status": "pending" },
    { "name": "Rejects invalid email format", "file": "tests/user.test.js", "status": "pending" },
    { "name": "Hashes password before storing", "file": "tests/user.test.js", "status": "pending" },
    { "name": "Returns 409 for duplicate email", "file": "tests/user.test.js", "status": "pending" }
  ]
}
```

### Step 3: Implement and Update

```json
{
  "tests": [
    { "name": "Creates user with valid data", "file": "tests/user.test.js", "status": "pass" },
    { "name": "Rejects invalid email format", "file": "tests/user.test.js", "status": "pass" },
    { "name": "Hashes password before storing", "file": "tests/user.test.js", "status": "fail", "error": "bcrypt not imported" },
    { "name": "Returns 409 for duplicate email", "file": "tests/user.test.js", "status": "pending" }
  ]
}
```

---

## Configuring AI to Use tests.json

### Copilot Custom Instructions

```markdown
# copilot-instructions.md

## Test Tracking
- A `tests.json` file in the project root tracks all test status
- Before writing tests, check tests.json for existing tests
- After writing or running tests, update tests.json
- Format: { "tests": [{ "name": "...", "file": "...", "status": "pass|fail|pending" }] }
- Never create duplicate test entries
```

### CLAUDE.md

```markdown
# CLAUDE.md

## Test Tracking Protocol
1. Read tests.json at the start of any testing task
2. Check which tests are pending or failing
3. Work on failing tests first, then pending
4. Update tests.json after every test run
5. Include error messages for failing tests
6. Add notes for pending tests explaining what's needed
```

---

## Updating tests.json Automatically

### Script: Parse Test Results

```javascript
// scripts/update-tests-json.js
const { execSync } = require('child_process');
const fs = require('fs');

// Run tests with JSON reporter
const result = execSync('npx jest --json 2>/dev/null', { encoding: 'utf8' });
const jestOutput = JSON.parse(result);

const tests = [];
for (const suite of jestOutput.testResults) {
  for (const test of suite.testResults) {
    tests.push({
      name: test.fullName,
      file: suite.testFilePath.replace(process.cwd() + '/', ''),
      status: test.status === 'passed' ? 'pass' : test.status === 'failed' ? 'fail' : 'pending',
      duration: test.duration,
      ...(test.status === 'failed' && {
        error: test.failureMessages[0]?.split('\n')[0],
      }),
    });
  }
}

const tracking = {
  meta: {
    lastUpdated: new Date().toISOString(),
    totalTests: tests.length,
    passing: tests.filter(t => t.status === 'pass').length,
    failing: tests.filter(t => t.status === 'fail').length,
    pending: tests.filter(t => t.status === 'pending').length,
  },
  tests,
};

fs.writeFileSync('tests.json', JSON.stringify(tracking, null, 2));
console.log(`Updated tests.json: ${tracking.meta.passing} pass, ${tracking.meta.failing} fail, ${tracking.meta.pending} pending`);
```

### Pytest Version

```python
# scripts/update_tests_json.py
import json
import subprocess
import sys

result = subprocess.run(
    ["pytest", "--tb=line", "-q", "--json-report", "--json-report-file=-"],
    capture_output=True, text=True
)

report = json.loads(result.stdout)
tests = []

for test in report.get("tests", []):
    tests.append({
        "name": test["nodeid"],
        "file": test["nodeid"].split("::")[0],
        "status": {
            "passed": "pass",
            "failed": "fail",
            "skipped": "pending",
        }.get(test["outcome"], "pending"),
        "duration": test.get("duration"),
    })

tracking = {
    "meta": {
        "totalTests": len(tests),
        "passing": sum(1 for t in tests if t["status"] == "pass"),
        "failing": sum(1 for t in tests if t["status"] == "fail"),
    },
    "tests": tests,
}

with open("tests.json", "w") as f:
    json.dump(tracking, f, indent=2)
```

---

## Multi-Agent Test Coordination

When multiple agents work on the same project, tests.json prevents conflicts:

```
Agent A (auth):                    Agent B (orders):
─────────────                      ──────────────
1. Read tests.json                 1. Read tests.json
2. See: auth tests pending         2. See: order tests pending
3. Write auth tests                3. Write order tests
4. Update tests.json               4. Update tests.json
   status: pass                       status: pass
```

### Locking Pattern

```json
{
  "meta": {
    "lockedBy": "agent-auth-001",
    "lockedAt": "2024-03-15T10:30:00Z"
  },
  "tests": [...]
}
```

### Category-Based Coordination

```json
{
  "tests": [
    { "name": "...", "category": "auth", "assignedTo": "agent-1", "status": "in-progress" },
    { "name": "...", "category": "orders", "assignedTo": "agent-2", "status": "in-progress" },
    { "name": "...", "category": "payments", "assignedTo": null, "status": "pending" }
  ]
}
```

---

## Querying Test Status

### Quick Status Check

```bash
# Count by status
cat tests.json | python -c "
import json, sys
data = json.load(sys.stdin)
tests = data['tests']
for status in ['pass', 'fail', 'pending']:
    count = sum(1 for t in tests if t['status'] == status)
    print(f'{status}: {count}')
"
```

### Find Failing Tests

```bash
cat tests.json | python -c "
import json, sys
data = json.load(sys.stdin)
for t in data['tests']:
    if t['status'] == 'fail':
        print(f\"FAIL: {t['name']} ({t['file']})\")
        if 'error' in t:
            print(f\"  Error: {t['error']}\")
"
```
