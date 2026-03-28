# Flaky Test Detection

> Identifying and fixing non-deterministic tests — especially the subtle flakiness that AI-generated tests are prone to introducing.

---

## Why AI-Generated Tests Are Prone to Flakiness

AI models generate syntactically correct tests that often contain hidden non-determinism:

- **Timing assumptions** — AI uses `setTimeout`, `sleep`, or arbitrary delays
- **Shared state** — AI doesn't always isolate test data between tests
- **Order dependency** — tests pass in sequence but fail when randomized
- **Environment assumptions** — AI assumes specific ports, paths, or system state
- **Race conditions** — AI-generated async code doesn't always await properly

```
Flaky Test Impact:
──────────────────
1st flake:  "Hmm, probably a CI hiccup" → Re-run
5th flake:  "This test is unreliable" → Ignore failures  
20th flake: "Tests are useless" → Team stops trusting CI
```

---

## Common AI-Generated Flakiness Sources

### 1. setTimeout-Based Assertions

```javascript
// ❌ AI-generated: timing-dependent
test('shows notification after save', async () => {
  await saveDocument();
  setTimeout(() => {
    expect(screen.getByText('Saved!')).toBeInTheDocument();
  }, 100);  // Fails when system is slow
});

// ✅ Fixed: event-driven waiting
test('shows notification after save', async () => {
  await saveDocument();
  await waitFor(() => {
    expect(screen.getByText('Saved!')).toBeInTheDocument();
  });
});
```

### 2. Shared State Between Tests

```javascript
// ❌ AI-generated: shared database state
describe('UserService', () => {
  test('creates a user', async () => {
    await db.users.create({ email: 'test@example.com' });
    const user = await db.users.findByEmail('test@example.com');
    expect(user).toBeDefined();
  });

  test('lists all users', async () => {
    const users = await db.users.findAll();
    expect(users).toHaveLength(1);  // Depends on previous test!
  });
});

// ✅ Fixed: isolated test data
describe('UserService', () => {
  beforeEach(async () => {
    await db.users.deleteAll();  // Clean slate
  });

  test('creates a user', async () => {
    await db.users.create({ email: 'test@example.com' });
    const user = await db.users.findByEmail('test@example.com');
    expect(user).toBeDefined();
  });

  test('lists all users', async () => {
    await db.users.create({ email: 'a@test.com' });
    await db.users.create({ email: 'b@test.com' });
    const users = await db.users.findAll();
    expect(users).toHaveLength(2);  // Self-contained
  });
});
```

### 3. Environment Assumptions

```python
# ❌ AI-generated: assumes specific environment
def test_file_processing():
    result = process_file("/tmp/test-data.csv")  # Path may not exist
    assert result.rows == 100

def test_api_connection():
    response = requests.get("http://localhost:3000/health")  # Port may be taken
    assert response.status_code == 200

# ✅ Fixed: controlled environment
def test_file_processing(tmp_path):
    test_file = tmp_path / "test-data.csv"
    test_file.write_text(generate_csv_data(100))
    result = process_file(str(test_file))
    assert result.rows == 100

def test_api_connection(test_server):
    response = requests.get(f"{test_server.url}/health")
    assert response.status_code == 200
```

### 4. Race Conditions in Async Code

```javascript
// ❌ AI-generated: race condition
test('processes items concurrently', async () => {
  const results = [];
  items.forEach(async (item) => {
    const result = await processItem(item);
    results.push(result);
  });
  expect(results).toHaveLength(items.length);  // Empty! forEach doesn't await
});

// ✅ Fixed: proper async handling
test('processes items concurrently', async () => {
  const results = await Promise.all(
    items.map(item => processItem(item))
  );
  expect(results).toHaveLength(items.length);
});
```

### 5. Date/Time Sensitivity

```javascript
// ❌ AI-generated: time-dependent
test('formats relative time', () => {
  const post = { createdAt: new Date('2024-01-15T10:00:00Z') };
  expect(formatRelativeTime(post.createdAt)).toBe('3 months ago');
  // Fails next month!
});

// ✅ Fixed: controlled time
test('formats relative time', () => {
  jest.useFakeTimers();
  jest.setSystemTime(new Date('2024-04-15T10:00:00Z'));

  const post = { createdAt: new Date('2024-01-15T10:00:00Z') };
  expect(formatRelativeTime(post.createdAt)).toBe('3 months ago');

  jest.useRealTimers();
});
```

---

## Detection Strategies

### Strategy 1: Re-Run Failures

```yaml
# GitHub Actions: retry failed tests
- name: Run tests with retry
  uses: nick-fields/retry@v2
  with:
    timeout_minutes: 10
    max_attempts: 3
    command: npm test -- --ci
    # If test passes on retry, it's flaky
```

### Strategy 2: Repeated Execution

```bash
# Run tests multiple times to surface flakiness
# Jest
for i in {1..10}; do npx jest --ci 2>&1 | tail -1; done

# Pytest with repeat plugin
pip install pytest-repeat
pytest --count=10 tests/

# Vitest
for i in {1..10}; do npx vitest run 2>&1 | tail -1; done
```

### Strategy 3: Randomize Test Order

```bash
# Jest: randomize by default (v29+)
npx jest --randomize

# Pytest: install random order plugin
pip install pytest-random-order
pytest -p random_order

# If tests fail with random order, you have order dependency
```

### Strategy 4: Quarantine Flaky Tests

```javascript
// Tag flaky tests for separate tracking
describe('Payment Processing', () => {
  // Quarantined: flaky due to external payment API timing
  // Issue: #456, Owner: @alice, Date: 2024-03-15
  test.skip('processes refund within timeout', async () => {
    // ...
  });

  // Active tests
  test('validates payment amount', () => {
    // ...
  });
});
```

```yaml
# CI: Run quarantined tests separately
jobs:
  stable-tests:
    runs-on: ubuntu-latest
    steps:
      - run: npm test -- --testPathIgnorePatterns="quarantine"

  flaky-tests:
    runs-on: ubuntu-latest
    continue-on-error: true  # Don't block the build
    steps:
      - run: npm test -- --testPathPattern="quarantine"
```

---

## Flaky Test Dashboard

Track flake rates over time:

```markdown
## Flaky Test Tracker

| Test Name | File | Flake Rate | First Seen | AI-Generated? | Status |
|-----------|------|-----------|------------|---------------|--------|
| processes concurrent uploads | upload.test.js:45 | 15% | 2024-03-01 | Yes | Investigating |
| sends notification email | notify.test.js:23 | 8% | 2024-03-10 | Yes | Quarantined |
| renders dashboard charts | dashboard.test.js:67 | 3% | 2024-02-15 | No | Fixed |
```

### Automated Flake Tracking

```javascript
// scripts/track-flakes.js
const fs = require('fs');

function updateFlakeTracker(testResults) {
  const tracker = JSON.parse(fs.readFileSync('flake-tracker.json', 'utf8'));

  for (const result of testResults) {
    if (result.status === 'failed' && result.retrySucceeded) {
      const key = `${result.file}:${result.name}`;
      if (!tracker[key]) {
        tracker[key] = { count: 0, total: 0, firstSeen: new Date().toISOString() };
      }
      tracker[key].count++;
      tracker[key].total++;
      tracker[key].flakeRate = (tracker[key].count / tracker[key].total * 100).toFixed(1);
    }
  }

  fs.writeFileSync('flake-tracker.json', JSON.stringify(tracker, null, 2));
}
```

---

## Fixing Flaky Tests with AI

AI is actually good at fixing flakiness when given the right context:

```markdown
# Effective prompt for flaky test fixes
This test is flaky — it passes ~85% of the time:

```javascript
test('updates user profile', async () => {
  await updateProfile({ name: 'New Name' });
  const profile = await getProfile(userId);
  expect(profile.name).toBe('New Name');
});
```

Failure message when it fails:
"Expected 'New Name', received 'Old Name'"

The test uses a real database. The update is async.

Fix the flakiness without reducing test value.
Do NOT add arbitrary sleep/delay.
```

### AI Fix Pattern

```javascript
// AI-generated fix: proper async waiting
test('updates user profile', async () => {
  await updateProfile({ name: 'New Name' });

  // Wait for the async update to propagate
  await expect(async () => {
    const profile = await getProfile(userId);
    expect(profile.name).toBe('New Name');
  }).toPass({ timeout: 5000, intervals: [100, 200, 500] });
});
```

---

## Prevention: Instruction File Rules

```markdown
# Add to copilot-instructions.md or CLAUDE.md

## Flaky Test Prevention
- Never use setTimeout/sleep in tests — use event-driven waiting
- Always isolate test data — use beforeEach cleanup or factories
- Never depend on test execution order
- Use fake timers for time-dependent tests
- Use dynamic ports for server tests
- Mock external services — don't depend on network
- Use `waitFor` or retry patterns for async assertions
- Every test must pass when run in isolation
```

---

## Flakiness Prevention Checklist

```markdown
## Before Merging AI-Generated Tests
- [ ] No setTimeout/sleep in assertions
- [ ] No shared mutable state between tests
- [ ] No hardcoded ports, paths, or timestamps
- [ ] All async operations properly awaited
- [ ] Tests pass when run individually (`--testNamePattern`)
- [ ] Tests pass with randomized order (`--randomize`)
- [ ] Tests pass 10/10 times in repeated execution
- [ ] External dependencies are mocked or containerized
```

---

## Next Steps

- [Regression Testing Strategy](regression-testing.md) — the broader regression prevention framework
- [Test Maintenance](test-maintenance.md) — keeping tests healthy over time
- [Anti-Pattern: Testing to Pass](../10-anti-patterns/testing-to-pass.md) — another common AI testing pitfall
- [Troubleshooting AI Testing](../11-practical/troubleshooting.md) — fixing common AI test generation problems

---

*Part of the [Testing Strategy with AI](../README.md) series.*
