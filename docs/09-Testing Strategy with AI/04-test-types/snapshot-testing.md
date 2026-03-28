# Snapshot and Visual Testing

> UI regression detection — automatically capturing component output and flagging unexpected changes, with AI helping generate meaningful snapshots and review visual diffs.

## Snapshot Testing Basics

Snapshot tests capture the rendered output of a component and compare it against a stored reference. If the output changes, the test fails until you approve the update.

```typescript
// Jest snapshot test
import { render } from '@testing-library/react';
import { UserCard } from './UserCard';

it('should render user card correctly', () => {
  const { container } = render(
    <UserCard
      user={{ name: 'Alice', email: 'alice@example.com', role: 'admin' }}
    />
  );
  expect(container).toMatchSnapshot();
});
```

First run creates a `.snap` file:

```
// Jest Snapshot v1

exports[`should render user card correctly 1`] = `
<div class="user-card">
  <h3>Alice</h3>
  <p>alice@example.com</p>
  <span class="badge badge-admin">admin</span>
</div>
`;
```

Subsequent runs compare the current output against this stored snapshot.

## When Snapshots Help vs Hurt

### ✅ Snapshots Help When

- **Detecting unintended changes** — A CSS class rename accidentally breaks a component
- **Documenting component output** — The snapshot serves as a readable reference for what the component renders
- **Testing complex rendered output** — Components with conditional rendering, lists, or nested structures
- **Catching regressions during refactors** — Changing internal logic shouldn't change the output

### ❌ Snapshots Hurt When

- **Snapshots are too large** — A 500-line snapshot is impossible to review meaningfully
- **Snapshots change frequently** — If every PR updates snapshots, they add noise instead of signal
- **Approving without reviewing** — Running `jest --updateSnapshot` without checking what changed
- **Testing implementation details** — Snapshots of internal component structure that change with every refactor

## Best Practice: Small, Focused Snapshots

```typescript
// ❌ Bad: Entire page snapshot — too large to review
it('should render the settings page', () => {
  const { container } = render(<SettingsPage />);
  expect(container).toMatchSnapshot(); // 200+ lines of HTML
});

// ✅ Good: Focused snapshot of specific component output
it('should render permission badges for admin user', () => {
  const { container } = render(
    <PermissionBadges roles={['admin', 'editor']} />
  );
  expect(container).toMatchSnapshot();
  // Snapshot is ~5 lines, easy to review
});

// ✅ Better: Inline snapshot for very small output
it('should format user display name', () => {
  const result = formatDisplayName({ first: 'Alice', last: 'Smith', title: 'Dr.' });
  expect(result).toMatchInlineSnapshot(`"Dr. Alice Smith"`);
});
```

## AI-Generated Snapshots for Complex Components

AI can help create meaningful snapshot tests by identifying the important rendering variations:

### Prompt for AI

```
Generate snapshot tests for <DataTable> component.

The component accepts:
- data: Array of objects
- columns: Column definitions with header, accessor, optional formatter
- sortable: boolean
- emptyMessage: string

Generate snapshots for these variations:
1. Normal data (3 rows, 3 columns)
2. Empty data (should show emptyMessage)
3. With custom formatters (currency, date)
4. Sorted by different columns

Keep each snapshot focused — don't snapshot the entire table for
every variation. Snapshot just the part that's different.
```

### AI-Generated Result

```typescript
import { render, screen } from '@testing-library/react';
import { DataTable } from './DataTable';

const columns = [
  { header: 'Name', accessor: 'name' },
  { header: 'Price', accessor: 'price', formatter: (v: number) => `$${v.toFixed(2)}` },
  { header: 'Date', accessor: 'date' },
];

const sampleData = [
  { name: 'Widget A', price: 29.99, date: '2024-01-15' },
  { name: 'Widget B', price: 49.50, date: '2024-02-20' },
  { name: 'Widget C', price: 9.99, date: '2024-03-10' },
];

describe('DataTable snapshots', () => {
  it('should render table body with formatted data', () => {
    const { container } = render(
      <DataTable data={sampleData} columns={columns} />
    );
    const tbody = container.querySelector('tbody');
    expect(tbody).toMatchSnapshot();
  });

  it('should render empty state message', () => {
    const { container } = render(
      <DataTable data={[]} columns={columns} emptyMessage="No items found" />
    );
    expect(container.querySelector('.empty-state')).toMatchSnapshot();
  });

  it('should render column headers with sort indicators when sortable', () => {
    const { container } = render(
      <DataTable data={sampleData} columns={columns} sortable />
    );
    const thead = container.querySelector('thead');
    expect(thead).toMatchSnapshot();
  });
});
```

## Visual Regression Testing

Visual regression testing goes beyond DOM snapshots — it captures **screenshots** and compares them pixel by pixel.

### Tools

| Tool | Type | Approach |
|------|------|----------|
| [Percy](https://percy.io/) | Cloud service | Captures screenshots in CI, provides visual diff review UI |
| [Chromatic](https://www.chromatic.com/) | Cloud service | Built for Storybook, captures every story as a visual test |
| [Playwright screenshots](https://playwright.dev/docs/screenshots) | Built-in | `await expect(page).toHaveScreenshot()` |
| [BackstopJS](https://github.com/garris/BackstopJS) | Open source | Config-driven visual regression testing |

### Playwright Visual Comparison

```typescript
import { test, expect } from '@playwright/test';

test('homepage should match visual snapshot', async ({ page }) => {
  await page.goto('/');
  await expect(page).toHaveScreenshot('homepage.png', {
    maxDiffPixels: 100, // Allow minor anti-aliasing differences
  });
});

test('login form should match visual snapshot', async ({ page }) => {
  await page.goto('/login');
  const form = page.getByTestId('login-form');
  await expect(form).toHaveScreenshot('login-form.png');
});

test('dashboard renders correctly in dark mode', async ({ page }) => {
  await page.goto('/dashboard');
  await page.emulateMedia({ colorScheme: 'dark' });
  await expect(page).toHaveScreenshot('dashboard-dark.png');
});
```

### Storybook + Chromatic

```typescript
// Button.stories.tsx — Each story becomes a visual test
import type { Meta, StoryObj } from '@storybook/react';
import { Button } from './Button';

const meta: Meta<typeof Button> = {
  component: Button,
};
export default meta;

type Story = StoryObj<typeof Button>;

export const Primary: Story = {
  args: { variant: 'primary', children: 'Click Me' },
};

export const Disabled: Story = {
  args: { variant: 'primary', children: 'Click Me', disabled: true },
};

export const Loading: Story = {
  args: { variant: 'primary', children: 'Click Me', loading: true },
};

// Chromatic captures a screenshot of each story
// and flags visual changes in PRs
```

## Common Snapshot Anti-Patterns

### 1. Snapshots That Test Nothing

```typescript
// ❌ Developer runs: jest --updateSnapshot
// Without reviewing what changed — the "approve everything" problem
it('renders', () => {
  const { container } = render(<ComplexForm />);
  expect(container).toMatchSnapshot();
});
```

**Fix**: Review snapshot diffs in PRs just as carefully as code diffs. Use focused snapshots that are small enough to actually review.

### 2. Large Snapshots That Are Impossible to Review

```typescript
// ❌ This snapshot is 300 lines — nobody will review it
it('should render the entire admin panel', () => {
  const { container } = render(<AdminPanel />);
  expect(container).toMatchSnapshot();
});

// ✅ Snapshot specific sections
it('should render the sidebar navigation', () => {
  const { container } = render(<AdminPanel />);
  expect(container.querySelector('.sidebar-nav')).toMatchSnapshot();
});
```

### 3. Snapshots with Unstable Data

```typescript
// ❌ Snapshot changes every run because of timestamps and IDs
it('should render order', () => {
  const { container } = render(<Order order={realOrder} />);
  expect(container).toMatchSnapshot();
  // Snapshot includes: "Order #a3f2b1c9 — Jan 15, 2024 10:32:04 AM"
});

// ✅ Use stable test data
it('should render order', () => {
  const stableOrder = {
    id: 'ORDER-001',
    date: new Date('2024-01-15T00:00:00Z'),
    items: [{ name: 'Widget', price: 29.99 }],
  };
  const { container } = render(<Order order={stableOrder} />);
  expect(container).toMatchSnapshot();
});
```

## Snapshot Testing Decision Guide

```
Is the output small (< 20 lines)?
  ├─ Yes → Snapshot test is a good fit
  │         Consider inline snapshots for < 5 lines
  └─ No  → Can you snapshot a focused subset?
            ├─ Yes → Snapshot the subset
            └─ No  → Use assertion-based tests instead
                     or visual regression testing
```

## Next Steps

- [E2E Testing](./e2e-testing.md) — Combine visual testing with full workflow validation using Playwright
- [Unit Testing](./unit-testing.md) — When assertion-based tests are better than snapshots
- [Test Quality Metrics](../05-coverage-and-quality/test-quality-metrics.md) — Measure whether your snapshot tests provide real value
