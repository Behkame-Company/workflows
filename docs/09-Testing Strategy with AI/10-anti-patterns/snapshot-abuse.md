# Anti-Pattern: Snapshot Abuse

> When AI generates massive snapshots that nobody reviews — catching changes but not bugs, creating maintenance burden without testing value.

---

## The Problem

AI loves snapshot tests because they're easy to generate: render a component, serialize the output, done. But this creates a false sense of coverage:

- **500-line snapshots** that no human will ever read
- **Auto-approved updates** because "the snapshot changed, just update it"
- **Change detectors, not bug detectors** — they tell you *something* changed, not whether it's wrong

```
AI generates snapshot:      Developer sees snapshot failure:
──────────────────────      ────────────────────────────────
"Here's a test!" ✅          "Snapshot outdated"
                             → git diff: 200 lines changed
                             → "Looks fine, update snapshot"
                             → npx jest --updateSnapshot
                             → Bug shipped ✅
```

---

## How AI Creates Snapshot Abuse

### The Typical AI Pattern

```javascript
// AI generates this when asked to "write tests for the Dashboard component"
import { render } from '@testing-library/react';
import Dashboard from './Dashboard';

test('renders dashboard', () => {
  const { container } = render(
    <Dashboard
      user={{ name: 'Alice', role: 'admin' }}
      stats={{ revenue: 50000, users: 1200 }}
      notifications={[{ id: 1, message: 'Welcome' }]}
    />
  );
  expect(container).toMatchSnapshot();
});
```

### What the Snapshot File Looks Like

```javascript
// __snapshots__/Dashboard.test.js.snap
exports[`renders dashboard 1`] = `
<div class="dashboard">
  <header class="dashboard-header">
    <div class="user-info">
      <span class="user-name">Alice</span>
      <span class="user-role">admin</span>
    </div>
    <nav class="main-nav">
      <a href="/home">Home</a>
      <a href="/reports">Reports</a>
      <a href="/settings">Settings</a>
    </nav>
  </header>
  <main class="dashboard-content">
    <div class="stats-grid">
      <div class="stat-card">
        <h3>Revenue</h3>
        <span class="stat-value">$50,000</span>
      </div>
      <div class="stat-card">
        <h3>Users</h3>
        <span class="stat-value">1,200</span>
      </div>
    </div>
    <div class="notifications">
      <div class="notification-item">
        <p>Welcome</p>
      </div>
    </div>
  </main>
  <!-- ... 300 more lines ... -->
</div>
`;
```

**Nobody will meaningfully review this.** When it breaks, the response is always `--updateSnapshot`.

---

## When Snapshots ARE Useful

Snapshots work well for **small, focused, stable** output:

```javascript
// ✅ Good: Small, focused snapshot
test('renders error badge with correct styling', () => {
  const { container } = render(<Badge type="error" text="Failed" />);
  expect(container.firstChild).toMatchInlineSnapshot(`
    <span class="badge badge-error">
      Failed
    </span>
  `);
});

// ✅ Good: Serialized data structure
test('formats API response correctly', () => {
  const response = formatUserResponse(testUser);
  expect(response).toMatchInlineSnapshot(`
    {
      "id": 1,
      "displayName": "Alice Smith",
      "role": "admin",
    }
  `);
});
```

### Snapshot Decision Matrix

| Scenario | Use Snapshot? | Why |
|----------|:------------:|-----|
| Small UI component (< 20 lines) | ✅ | Reviewable, stable |
| Inline snapshot (in test file) | ✅ | Visible during review |
| Full page render | ❌ | Too large to review |
| API response format | ✅ | Small, structured |
| Entire component tree | ❌ | Changes too often |
| Error messages | ✅ | Important to be exact |
| Dynamic content | ❌ | Breaks constantly |

---

## Detection: How to Spot Snapshot Abuse

### Signal 1: Snapshot Files Larger Than Source Files

```bash
# Compare snapshot size to source size
find . -name "*.snap" -exec wc -l {} + | sort -rn | head -10

# If snapshot files are 2-10x the size of source files, you have abuse
```

### Signal 2: Frequent "Update Snapshot" Commits

```bash
# Check git history for snapshot updates
git log --oneline --all | grep -i "update.*snapshot\|snapshot.*update" | wc -l

# More than a few per month is a warning sign
```

### Signal 3: Snapshots With No Other Assertions

```javascript
// ❌ Entire test is just a snapshot — no behavioral assertions
test('renders user profile', () => {
  const { container } = render(<UserProfile user={testUser} />);
  expect(container).toMatchSnapshot();
  // That's it. No behavioral testing at all.
});
```

### Signal 4: Nobody Reviews Snapshot Diffs

Ask your team: "When a snapshot test fails in a PR, do you read the full snapshot diff or just update it?" If the answer is "update it," the snapshots aren't providing value.

---

## The Fix: Replace with Targeted Assertions

### Before: Snapshot Abuse

```javascript
// ❌ 500-line snapshot nobody reads
test('renders dashboard', () => {
  const { container } = render(<Dashboard user={admin} stats={mockStats} />);
  expect(container).toMatchSnapshot();
});
```

### After: Behavioral Assertions

```javascript
// ✅ Tests specific, important behaviors
describe('Dashboard', () => {
  test('displays user name in header', () => {
    render(<Dashboard user={admin} stats={mockStats} />);
    expect(screen.getByText('Alice')).toBeInTheDocument();
  });

  test('shows revenue formatted as currency', () => {
    render(<Dashboard user={admin} stats={{ revenue: 50000 }} />);
    expect(screen.getByText('$50,000')).toBeInTheDocument();
  });

  test('shows admin navigation for admin users', () => {
    render(<Dashboard user={admin} stats={mockStats} />);
    expect(screen.getByRole('link', { name: 'Settings' })).toBeInTheDocument();
  });

  test('hides admin navigation for regular users', () => {
    render(<Dashboard user={regularUser} stats={mockStats} />);
    expect(screen.queryByRole('link', { name: 'Settings' })).not.toBeInTheDocument();
  });

  test('displays notification count badge', () => {
    render(<Dashboard user={admin} stats={mockStats} notifications={[n1, n2]} />);
    expect(screen.getByText('2')).toBeInTheDocument();
  });
});
```

### When to Keep Small Snapshots

```javascript
// ✅ Inline snapshot for small, critical UI elements
test('error alert has correct structure and aria role', () => {
  render(<Alert type="error" message="Payment failed" />);
  expect(screen.getByRole('alert')).toMatchInlineSnapshot(`
    <div
      class="alert alert-error"
      role="alert"
    >
      <svg class="alert-icon" />
      <span>Payment failed</span>
    </div>
  `);
});
```

---

## Snapshot Hygiene Rules

### Size Limits

```javascript
// jest.config.js — enforce snapshot size limits with custom serializer
module.exports = {
  snapshotSerializers: ['./test-utils/snapshot-size-check'],
};

// test-utils/snapshot-size-check.js
module.exports = {
  test(val) {
    return typeof val === 'string';
  },
  serialize(val) {
    if (val.length > 2000) {
      throw new Error(
        `Snapshot too large (${val.length} chars). ` +
        'Use targeted assertions instead of full-component snapshots.'
      );
    }
    return val;
  },
};
```

### PR Review Rule

```markdown
## Snapshot Review Policy
- Inline snapshots (< 30 lines): review in PR diff
- External snapshots (> 30 lines): REJECT — replace with assertions
- New snapshot files: require justification in PR description
- "Update snapshot" commits: require explanation of what changed and why
```

---

## Configuring AI to Avoid Snapshot Abuse

```markdown
# copilot-instructions.md or CLAUDE.md

## Snapshot Test Rules
- Do NOT generate full-component snapshot tests
- Use inline snapshots ONLY for small elements (< 20 lines)
- Prefer behavioral assertions: `getByText`, `getByRole`, `toBeInTheDocument`
- Test what the user sees and does, not the HTML structure
- If you must use a snapshot, use `toMatchInlineSnapshot()` so it's visible in the test file
- Never snapshot an entire page or complex component
```

---

## Migration: From Snapshots to Assertions

```markdown
## Snapshot Migration Plan
1. Identify snapshot files > 100 lines
2. For each large snapshot test:
   a. List the 3-5 most important behaviors it captures
   b. Write behavioral assertions for those behaviors
   c. Delete the snapshot test
   d. Run the new tests — verify they pass
3. Keep only small, inline snapshots for stable, small components
4. Add CI rule: reject new snapshot files > 50 lines
```

---

## Next Steps

- [Anti-Pattern: Testing to Pass](testing-to-pass.md) — another way tests provide false confidence
- [Anti-Pattern: Skipping Verification](skipping-verification.md) — trusting without testing
- [Snapshot and Visual Testing](../04-test-types/snapshot-testing.md) — when snapshots ARE the right approach
- [Test Quality Metrics](../05-coverage-and-quality/test-quality-metrics.md) — measuring real test value

---

*Part of the [Testing Strategy with AI](../README.md) series.*
