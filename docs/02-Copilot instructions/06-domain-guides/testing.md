# Domain Guide: Testing

> Instruction patterns for unit tests, integration tests, and E2E tests.

---

## Sample: testing.instructions.md

```yaml
---
applyTo: "**/*.{test,spec}.{ts,tsx}"
---
```

```markdown
# Testing Instructions

## Framework
- Unit tests: Vitest
- Component tests: Vitest + React Testing Library
- E2E tests: Playwright (in `e2e/` directory)
- API tests: Vitest + supertest

## Test Structure
- Use `describe` blocks for grouping by module/component
- Use `it` blocks with behavior descriptions: `it('should [do X] when [Y]')`
- One test per behavior — don't test multiple things in one `it` block

## Naming
- Test file: `[Module].test.ts` (co-located with source file)
- Describe block: `describe('ModuleName', () => {})`
- Test case: `it('should return filtered users when search query provided')`

## Mocking
- Mock external services (API calls, databases) — never hit real services
- Use `vi.mock()` for module-level mocks
- Use `vi.fn()` for individual function mocks
- Reset mocks in `beforeEach`: `vi.resetAllMocks()`
- Prefer mocking at the boundary (HTTP, database) over internal functions

## Assertions
- Use specific matchers: `toEqual` over `toBe` for objects
- Test error cases explicitly: `expect(() => fn()).toThrow()`
- Use `toHaveBeenCalledWith()` for verifying mock calls
- Prefer `getByRole` > `getByText` > `getByTestId` for component queries

## What to Test
- Business logic in `src/lib/` and `src/services/` — ALWAYS test
- React components — test user interactions and rendered output
- API routes — test request/response cycle with various inputs
- Edge cases: null, undefined, empty strings, boundary values, error conditions

## What NOT to Test
- Constants and static values
- Type definitions
- Third-party library internals
- Implementation details (private methods, internal state)
- CSS classes or styling (use visual testing tools instead)

## Coverage
- Minimum 80% for business logic (`src/lib/`, `src/services/`)
- Minimum 60% for components (focus on interactive behavior)
- No coverage requirement for config files, types, or generated code

## Do Not
- Never test implementation details (test behavior, not internals)
- Never hardcode dates — use `vi.setSystemTime()` for time-dependent tests
- Never leave `test.only` or `describe.only` in committed code
- Never import from test files in production code
```

---

## Component Testing Pattern

```markdown
## React Component Tests
When testing React components:

1. Render with necessary providers (QueryClientProvider, ThemeProvider, etc.)
2. Find elements by accessible queries (getByRole, getByLabelText)
3. Simulate user interactions (click, type, select)
4. Assert on visible changes (text appears, element is disabled, etc.)

Example pattern:
```typescript
import { render, screen } from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import { UserProfile } from './UserProfile'

describe('UserProfile', () => {
  it('should display user name after loading', async () => {
    // Arrange
    render(<UserProfile userId="123" />)

    // Act — wait for async data
    const name = await screen.findByText('Jane Doe')

    // Assert
    expect(name).toBeInTheDocument()
  })
})
```

---

## E2E Testing Instructions

```yaml
---
applyTo: "e2e/**/*.{ts,spec.ts}"
---
```

```markdown
# E2E Test Instructions

- Use Playwright with TypeScript
- Tests in `e2e/` directory mirror user workflows
- Each test file covers one user journey
- Use Page Object pattern for complex pages
- Tests must be independent (no shared state between tests)
- Use `test.describe` for logical grouping
- Always wait for elements before interacting: `await page.waitForSelector()`
- Take screenshots on failure (configured in playwright.config.ts)

## Common Patterns
- Login: use `storageState` from auth setup, don't repeat login in every test
- API mocking: use `page.route()` for controlling server responses
- Assertions: use Playwright's auto-waiting assertions (`expect(locator).toBeVisible()`)
```
