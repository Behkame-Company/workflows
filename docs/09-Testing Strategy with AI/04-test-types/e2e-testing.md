# E2E Testing with AI

> Full workflow validation with AI assistance — testing complete user journeys through the real application stack, from browser interaction to database state.

## What E2E Tests Verify

End-to-end tests exercise the entire application as a real user would. They are the final safety net before production:

- **User workflows complete successfully** — signup, login, purchase, etc.
- **UI elements render and respond** — buttons click, forms submit, pages navigate
- **Frontend ↔ Backend integration** — API calls work with real data
- **Critical business flows don't break** — the paths that lose money if they fail

## Playwright + AI for E2E Test Generation

[Playwright](https://playwright.dev/) is the modern standard for E2E testing. AI tools excel at generating Playwright tests because they understand the pattern well.

### Copilot Coding Agent and Playwright

The Copilot coding agent has a **Playwright MCP server enabled by default**, which means it can:

- Run Playwright tests directly during code generation
- Take screenshots and analyze visual output
- Debug failing E2E tests by inspecting actual page state
- Iterate on tests until they pass against the real application

This makes the coding agent especially effective for E2E test creation — it can write the test, run it, see the failure, and fix it in a loop.

## Writing E2E Test Scenarios as User Stories

The best way to prompt AI for E2E tests is to describe them as user stories:

```
Write Playwright E2E tests for these user stories:

1. As a new user, I can sign up with email and password,
   see a welcome page, and receive a confirmation email link.

2. As a returning user, I can log in, see my dashboard
   with recent orders, and click an order to see details.

3. As a shopper, I can search for a product, add it to cart,
   proceed to checkout, enter payment info, and see
   an order confirmation with an order number.

Use Page Object Model pattern.
Base URL: http://localhost:3000
Test user credentials: test@example.com / TestPass123!
```

## Page Object Model with AI

AI can generate page objects from UI descriptions, keeping tests maintainable:

```typescript
// pages/login.page.ts — AI-generated page object
import { Page, Locator } from '@playwright/test';

export class LoginPage {
  readonly page: Page;
  readonly emailInput: Locator;
  readonly passwordInput: Locator;
  readonly submitButton: Locator;
  readonly errorMessage: Locator;
  readonly forgotPasswordLink: Locator;

  constructor(page: Page) {
    this.page = page;
    this.emailInput = page.getByLabel('Email');
    this.passwordInput = page.getByLabel('Password');
    this.submitButton = page.getByRole('button', { name: 'Sign In' });
    this.errorMessage = page.getByRole('alert');
    this.forgotPasswordLink = page.getByRole('link', { name: 'Forgot password?' });
  }

  async goto() {
    await this.page.goto('/login');
  }

  async login(email: string, password: string) {
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    await this.submitButton.click();
  }

  async expectError(message: string) {
    await expect(this.errorMessage).toContainText(message);
  }
}
```

```typescript
// pages/dashboard.page.ts
import { Page, Locator } from '@playwright/test';

export class DashboardPage {
  readonly page: Page;
  readonly welcomeHeading: Locator;
  readonly recentOrders: Locator;
  readonly searchInput: Locator;

  constructor(page: Page) {
    this.page = page;
    this.welcomeHeading = page.getByRole('heading', { name: /welcome/i });
    this.recentOrders = page.getByTestId('recent-orders');
    this.searchInput = page.getByPlaceholder('Search products...');
  }

  async expectLoaded() {
    await expect(this.welcomeHeading).toBeVisible();
  }

  async getOrderCount() {
    return await this.recentOrders.locator('[data-testid="order-row"]').count();
  }
}
```

## Complete E2E Test Example

```typescript
// tests/checkout-flow.spec.ts
import { test, expect } from '@playwright/test';
import { LoginPage } from '../pages/login.page';
import { DashboardPage } from '../pages/dashboard.page';
import { ProductPage } from '../pages/product.page';
import { CartPage } from '../pages/cart.page';
import { CheckoutPage } from '../pages/checkout.page';

test.describe('Checkout Flow', () => {
  let loginPage: LoginPage;
  let dashboard: DashboardPage;

  test.beforeEach(async ({ page }) => {
    loginPage = new LoginPage(page);
    dashboard = new DashboardPage(page);

    // Login before each test
    await loginPage.goto();
    await loginPage.login('test@example.com', 'TestPass123!');
    await dashboard.expectLoaded();
  });

  test('complete purchase from search to confirmation', async ({ page }) => {
    // Search for a product
    const productPage = new ProductPage(page);
    await dashboard.searchInput.fill('Wireless Keyboard');
    await dashboard.searchInput.press('Enter');

    // Select first result
    await page.getByTestId('product-card').first().click();
    await expect(productPage.productTitle).toBeVisible();

    // Add to cart
    await productPage.addToCartButton.click();
    await expect(productPage.cartBadge).toHaveText('1');

    // Go to cart
    const cartPage = new CartPage(page);
    await productPage.cartBadge.click();
    await expect(cartPage.cartItems).toHaveCount(1);

    // Proceed to checkout
    const checkoutPage = new CheckoutPage(page);
    await cartPage.checkoutButton.click();

    // Fill payment details
    await checkoutPage.fillShipping({
      address: '123 Test St',
      city: 'Testville',
      zip: '12345',
    });
    await checkoutPage.fillPayment({
      cardNumber: '4242424242424242',
      expiry: '12/25',
      cvc: '123',
    });

    // Complete order
    await checkoutPage.placeOrderButton.click();

    // Verify confirmation
    await expect(page.getByRole('heading', { name: 'Order Confirmed' })).toBeVisible();
    const orderNumber = await page.getByTestId('order-number').textContent();
    expect(orderNumber).toMatch(/^ORD-\d+$/);
  });

  test('show error for invalid payment card', async ({ page }) => {
    // Navigate directly to checkout with items in cart
    await page.goto('/cart');
    const cartPage = new CartPage(page);
    await cartPage.checkoutButton.click();

    const checkoutPage = new CheckoutPage(page);
    await checkoutPage.fillShipping({
      address: '123 Test St',
      city: 'Testville',
      zip: '12345',
    });
    await checkoutPage.fillPayment({
      cardNumber: '4000000000000002', // Decline test card
      expiry: '12/25',
      cvc: '123',
    });

    await checkoutPage.placeOrderButton.click();

    await expect(page.getByRole('alert')).toContainText('Payment declined');
    // User stays on checkout page
    await expect(page).toHaveURL(/\/checkout/);
  });
});
```

## Handling Flaky E2E Tests

E2E tests are inherently more prone to flakiness than unit or integration tests. Common causes and fixes:

| Flaky Pattern | Fix |
|---------------|-----|
| Element not yet visible | Use `await expect(locator).toBeVisible()` before interacting |
| Animation or transition in progress | Use `await page.waitForLoadState('networkidle')` or wait for specific element |
| Race condition with API calls | Use `await page.waitForResponse('**/api/orders')` to wait for specific network calls |
| Test data from previous run | Use `beforeEach` to reset state via API or database |
| Hardcoded timeouts (`page.waitForTimeout`) | Replace with explicit waits for conditions |

### Playwright Built-in Resilience

```typescript
// ✅ Playwright auto-waits for elements — no manual waits needed
await page.getByRole('button', { name: 'Submit' }).click();

// ✅ Use web-first assertions that auto-retry
await expect(page.getByText('Success')).toBeVisible();

// ❌ Avoid manual timeouts
await page.waitForTimeout(3000); // Fragile — too short or too slow
```

## When to Use E2E vs Integration Tests

| Use E2E When | Use Integration When |
|-------------|---------------------|
| Testing critical user workflows | Testing API contracts and service boundaries |
| Verifying UI + backend together | Testing database queries and constraints |
| Checking cross-page navigation | Verifying business logic across services |
| Validating authentication flows | Testing error handling between components |
| Smoke testing a deployment | Testing performance of specific operations |

**Rule of thumb**: If the bug would be visible to a user in the browser, E2E catches it. If it's an internal contract violation, integration catches it.

## Playwright Configuration for AI Workflows

```typescript
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests/e2e',
  timeout: 30000,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,

  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',     // Capture trace on failure for debugging
    screenshot: 'only-on-failure',
  },

  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
    { name: 'firefox', use: { ...devices['Desktop Firefox'] } },
    { name: 'mobile', use: { ...devices['Pixel 5'] } },
  ],

  webServer: {
    command: 'npm run start',
    port: 3000,
    reuseExistingServer: !process.env.CI,
  },
});
```
