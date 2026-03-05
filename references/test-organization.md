# Test Organization

> **Skyline-specific**: See [skyline-conventions.md](skyline-conventions.md) for codebase-specific file naming, config patterns, test import requirements, and directory layout conventions that take precedence over generic guidance below.

## Table of Contents

1. [Configuration](#configuration)
2. [E2E Tests](#e2e-tests)
3. [API Tests](#api-tests)
4. [Directory Structure](#directory-structure)
5. [Tagging & Filtering](#tagging--filtering)

## Configuration

### Essential Configuration

> **Skyline note**: The Skyline codebase does NOT use `fullyParallel: true` or `webServer` in Playwright configs. End2EndTests run against a deployed environment. Web acceptance tests use a separate fake API server started independently. See [skyline-conventions.md](skyline-conventions.md) for actual config patterns.

```typescript
// playwright.config.ts
import { defineConfig, devices } from "@playwright/test";

export default defineConfig({
  testDir: "./tests",
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: [["html"], ["list"]],
  use: {
    baseURL: process.env.BASE_URL || "http://localhost:3000",
    trace: "on-first-retry",
    screenshot: "only-on-failure",
  },
  projects: [
    {
      name: "chromium",
      use: { ...devices["Desktop Chrome"] },
    },
  ],
});
```

## E2E Tests

Full user journey tests through the browser.

### Structure

```typescript
// tests/e2e/checkout.spec.ts
import { test, expect } from "@playwright/test";

test.describe("Checkout Flow", () => {
  test.beforeEach(async ({ page }) => {
    await page.goto("/products");
  });

  test("complete purchase as guest", async ({ page }) => {
    // Add to cart
    await page.getByRole("button", { name: "Add to Cart" }).first().click();
    await expect(page.getByTestId("cart-count")).toHaveText("1");

    // Go to checkout
    await page.getByRole("link", { name: "Cart" }).click();
    await page.getByRole("button", { name: "Checkout" }).click();

    // Fill shipping
    await page.getByLabel("Email").fill("guest@example.com");
    await page.getByLabel("Address").fill("123 Test St");
    await page.getByRole("button", { name: "Continue" }).click();

    // Payment
    await page.getByLabel("Card Number").fill("4242424242424242");
    await page.getByRole("button", { name: "Pay Now" }).click();

    // Confirmation
    await expect(page.getByRole("heading")).toHaveText("Order Confirmed");
  });

  test("apply discount code", async ({ page }) => {
    await page.getByRole("button", { name: "Add to Cart" }).first().click();
    await page.getByRole("link", { name: "Cart" }).click();

    await page.getByLabel("Discount Code").fill("SAVE10");
    await page.getByRole("button", { name: "Apply" }).click();

    await expect(page.getByText("10% discount applied")).toBeVisible();
  });
});
```

### Best Practices

- Test critical user journeys
- Keep tests independent
- Use realistic data
- Clean up test data in teardown

## API Tests

Test backend APIs without browser.

### API Mocking Patterns

For E2E tests that need to mock API responses:

```typescript
// Mock single endpoint
test("displays mocked users", async ({ page }) => {
  await page.route("**/api/users", (route) =>
    route.fulfill({
      status: 200,
      json: [{ id: 1, name: "Test User" }],
    }),
  );

  await page.goto("/users");
  await expect(page.getByText("Test User")).toBeVisible();
});

// Mock with different responses
test("handles API errors", async ({ page }) => {
  await page.route("**/api/users", (route) =>
    route.fulfill({
      status: 500,
      json: { error: "Server error" },
    }),
  );

  await page.goto("/users");
  await expect(page.getByText("Server error")).toBeVisible();
});

// Conditional mocking
test("mocks based on request", async ({ page }) => {
  await page.route("**/api/users", (route, request) => {
    if (request.method() === "GET") {
      route.fulfill({ json: [{ id: 1, name: "User" }] });
    } else {
      route.continue();
    }
  });
});

// Mock with delay (simulate slow network)
test("handles slow API", async ({ page }) => {
  await page.route("**/api/data", (route) =>
    route.fulfill({
      json: { data: "test" },
      delay: 2000, // 2 second delay
    }),
  );

  await page.goto("/dashboard");
  await expect(page.getByText("Loading...")).toBeVisible();
  await expect(page.getByText("test")).toBeVisible();
});
```

For advanced patterns (GraphQL mocking, HAR recording, request modification, network throttling), see **[network-advanced.md](network-advanced.md)**.

## Directory Structure

### Generic Structure

```
tests/
├── e2e/                    # End-to-end tests
│   ├── auth.spec.ts
│   ├── checkout.spec.ts
│   └── dashboard.spec.ts
├── api/                    # API tests
│   ├── users.spec.ts
│   └── products.spec.ts
├── fixtures/               # Custom fixtures
│   ├── auth.fixture.ts
│   └── api.fixture.ts
└── pages/                  # Page objects
    ├── login.page.ts
    └── dashboard.page.ts
```

> **Skyline note**: See [skyline-conventions.md](skyline-conventions.md) for actual Skyline directory layouts (End2EndTests tests/smoke + tests/userjourney, and per-app acceptance-tests/).

## Anti-Patterns to Avoid

| Anti-Pattern                          | Problem                            | Solution                  |
| ------------------------------------- | ---------------------------------- | ------------------------- |
| Long test files                       | Hard to maintain, slow to navigate | Split by feature, use POM |
| Tests depend on execution order       | Flaky, hard to debug               | Keep tests independent    |
| Testing multiple features in one test | Hard to debug failures             | One feature per test      |

## Related References

- **Projects**: See [projects-dependencies.md](projects-dependencies.md) for project-based filtering
- **Page Objects**: See [page-object-model.md](page-object-model.md) for organizing page interactions
- **Test Data**: See [fixtures-hooks.md](fixtures-hooks.md) for managing test data
- **Skyline Conventions**: See [skyline-conventions.md](skyline-conventions.md) for codebase-specific patterns

## Tagging & Filtering

### Using Tags

```typescript
test("user login @smoke @auth", async ({ page }) => {
  // ...
});

test("checkout flow @e2e @critical", async ({ page }) => {
  // ...
});

test.describe("API tests @api", () => {
  test("create user", async ({ request }) => {
    // ...
  });
});
```

### Skyline: ProductAreaTags

In the Skyline End2EndTests, tests are organized by product area using the `ProductAreaTags` enum. This is used alongside Playwright projects (smoke, userjourney) to categorize tests:

```typescript
import { ProductAreaTags } from '../shared/product-area-tags.js';

test.describe(`${ProductAreaTags.AssetManagement} Asset page`, () => {
  test('should display asset list', async ({ page }) => {
    // ...
  });
});
```

See [skyline-conventions.md](skyline-conventions.md) for the full list of product area tags and filtering conventions.

### Running Tagged Tests

```bash
# Run smoke tests
npx playwright test --grep @smoke

# Run all except slow tests
npx playwright test --grep-invert @slow

# Combine tags
npx playwright test --grep "@smoke|@critical"
```

For project-based filtering and advanced project configuration, see **[projects-dependencies.md](projects-dependencies.md)**.
