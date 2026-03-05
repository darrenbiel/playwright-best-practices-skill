# Skyline / SystemLink Conventions

This file documents Playwright testing conventions specific to the Skyline (SystemLink) codebase. These conventions take precedence over generic Playwright guidance when there is a conflict.

## Table of Contents

1. [Repository Layout](#repository-layout)
2. [Test Import Requirements](#test-import-requirements)
3. [File Naming](#file-naming)
4. [Configuration Patterns](#configuration-patterns)
5. [Page Object Conventions](#page-object-conventions)
6. [Locator Conventions for Custom Elements](#locator-conventions-for-custom-elements)
7. [Authentication](#authentication)
8. [Fixtures and Test Context](#fixtures-and-test-context)
9. [Tagging and Filtering](#tagging-and-filtering)
10. [Linting Rules](#linting-rules)
11. [ESM Import Requirements](#esm-import-requirements)
12. [Assertions Best Practices](#assertions-best-practices)

## Repository Layout

There are two distinct Playwright test suites in the repo:

### End2EndTests (E2E smoke + user journey tests)

```
End2EndTests/
├── playwright.config.ts
├── eslint.config.mjs
├── tests/
│   ├── smoke/              # Quick validation tests per feature area
│   │   ├── alarm-tests.ts
│   │   ├── assets-test.ts
│   │   └── ...
│   ├── userjourney/         # Multi-step user workflow tests
│   │   └── work-item-workflow-test.ts
│   └── shared/
│       ├── instrumentation/ # OpenTelemetry test wrappers
│       │   └── test.ts      # REQUIRED: import { test } from here
│       ├── page-objects/    # Page object classes
│       ├── services/        # API service wrappers
│       ├── utils/           # Utilities (enterprise-utils, http-context, etc.)
│       ├── config/          # Environment config
│       └── data/            # Test data
├── utils/
│   └── config-utils.ts      # Test filtering/disabling config
└── reporter/
    └── custom-reporter.ts   # CI custom reporter
```

### Web/Workspaces acceptance tests (per-app acceptance tests)

```
Web/Workspaces/<AppName>/acceptance-tests/
├── playwright.config.ts
├── Dockerfile
├── fake-api/               # Mock API server for isolated testing
├── tests/
│   ├── *.ts                # Test files
│   ├── config.ts           # Server URL config
│   ├── pages/              # Page object models
│   └── utils/              # Test utilities
└── tsconfig.json
```

Each Web app (Alarms, AssetManager, ControlRoom, TestInsights, etc.) has its own acceptance test suite that runs against a fake API server, testing the UI in isolation.

## Test Import Requirements

### End2EndTests: Use instrumented test import

**REQUIRED** (enforced by eslint): Import `test` from the shared instrumentation module, NOT directly from `@playwright/test`:

```typescript
// CORRECT - enables OpenTelemetry traces and metrics
import { test } from '../shared/instrumentation/test';
import { expect } from '@playwright/test';

// WRONG - blocked by eslint no-restricted-imports rule
import { test } from '@playwright/test';
```

The instrumented `test` automatically:
- Creates OpenTelemetry spans for each test execution
- Injects trace context headers into all page network requests
- Records test execution metrics (duration, status)
- Flushes telemetry per worker

### Web acceptance tests: Standard import

Web acceptance tests use the standard `@playwright/test` import:

```typescript
import { expect, test } from '@playwright/test';
```

## File Naming

### End2EndTests

- Test files: `*-test.ts` or `*-tests.ts` (NOT `.spec.ts`)
- Page objects: Match the element/page name with kebab-case (e.g., `alarms-page.ts`, `sl-grid.ts`)
- Services: `*-service.ts`

### Web acceptance tests

- Test files: `*-tests.ts` (consistent with End2EndTests)
- Page objects in `pages/` subdirectory

## Configuration Patterns

### End2EndTests playwright.config.ts

Key differences from generic Playwright guidance:

```typescript
const config: PlaywrightTestConfig = {
    testDir: './',
    timeout: 90 * 1000,          // 90s (longer than default 30s)
    expect: { timeout: 10000 },  // 10s expect timeout
    forbidOnly: isCI,
    retries: 1,                  // Always 1 retry (not 0 local / 2 CI)
    workers: process.env.TEST_AREA === 'smoke' ? 2 : 1,
    use: {
        actionTimeout: 20000,
        navigationTimeout: 60000,
        trace: isCI ? 'retain-on-failure' : 'on',
        video: 'retain-on-failure',
        screenshot: 'only-on-failure',
        ...devices['Desktop Chrome'],  // Chromium only, no multi-browser
    },
    projects: [
        { name: 'smoke', testMatch: ['tests/smoke/**/*test.ts', 'tests/smoke/**/*tests.ts'] },
        { name: 'userjourney', testMatch: ['tests/userjourney/**/*test.ts', 'tests/userjourney/**/*tests.ts'] },
    ],
};
```

Key points:
- **No `fullyParallel`**: Tests run file-by-file, not fully parallel
- **No `webServer`**: Tests run against deployed environments (not local dev servers)
- **No `baseURL`**: URLs are constructed from environment config
- **Chromium only**: No Firefox or WebKit projects
- **Projects are by test type** (smoke/userjourney), not by browser
- **Custom reporter** for CI: `./reporter/custom-reporter.ts`
- **Test filtering**: `grep`/`grepInvert` with `getDisabledTestsRegex()` for environment-specific exclusions

### Web acceptance test playwright.config.ts

Similar patterns with slightly different timeouts:
```typescript
const config: PlaywrightTestConfig = {
    testDir: './',
    testMatch: ['tests/*.ts'],
    timeout: 60 * 1000,
    expect: { timeout: 10000 - 15000 },
    retries: 0 - 1,             // Varies by app
    workers: 1 - 2,             // Varies by app
    use: {
        actionTimeout: 20000,
        navigationTimeout: 60000,
        trace: isCI ? 'retain-on-failure' : 'on',
        video: 'retain-on-failure',
        screenshot: 'only-on-failure',
    },
    projects: [{ name: 'chromium', use: { ...devices['Desktop Chrome'] } }],
};
```

## Page Object Conventions

### Construction pattern

Page objects take a `Page` in the constructor and expose `Locator` properties:

```typescript
import { Locator, Page } from '@playwright/test';

export class AlarmsPage {
    public readonly breadcrumbItem: Locator;
    public readonly alarmsTablePage: AlarmsTablePage;  // Sub-page objects for composition

    public constructor(page: Page) {
        this.breadcrumbItem = page.locator('nimble-breadcrumb-item');
        this.alarmsTablePage = new AlarmsTablePage(page);
    }
}
```

### Rules

- **DO**: Use instance-based page objects with `constructor(page: Page)`
- **DO**: Expose `Locator` objects as the API (not method return values like `isVisible()`)
- **DO**: Use composition — page objects contain other page objects for sub-sections
- **DO NOT**: Use static/singleton page object classes
- **DO NOT**: Create methods that return constant values (e.g., `async isElementVisible(): Promise<boolean>`). Instead expose the `Locator` so test code can use built-in assertions.
- **Action methods** (e.g., `clickForceClearButton()`) are fine for multi-step interactions.

### Naming

- Page objects for a page/view: `*-page.ts` with class `*Page` (e.g., `alarms-page.ts` → `AlarmsPage`)
- Page objects for a component: match the element name (e.g., `sl-grid.ts` → `SlGrid`)
- Place in directory matching the source app: `page-objects/alarms/`, `page-objects/systemlink-lib-angular/`

## Locator Conventions for Custom Elements

The UI uses custom web components (Nimble, SystemLink-specific) which affects locator strategy:

### Locator priority for this codebase

1. **CSS selectors for custom elements**: `page.locator('nimble-breadcrumb-item')`, `page.locator('sl-swif-header')`
2. **Role-based** when applicable: `page.getByRole('button', { name: 'Delete' })`
3. **Text-based with filter**: `page.locator('my-element', { hasText: 'ABC' })`
4. **Attribute-based**: `page.locator('[data-testid="..."]')`

### Key differences from generic Playwright guidance

- `getByRole()` is preferred for standard HTML elements but **CSS selectors are often necessary** for custom web components (`nimble-*`, `sl-*`) where ARIA roles may not be exposed
- Playwright **pierces shadow DOM by default** — no need for `shadowRoot()` chains
- Avoid selecting elements inside Nimble's shadow DOM when possible; interact with the component's public interface
- Use `.locator()` chaining across shadow boundaries: `elementLocator.locator('.class-1').locator('.class-2')`
- For text matching, prefer `{ hasText: 'ABC' }` or `RegExpUtils.forExactMatch('text')` over raw string selectors
- For Nimble text-field values, use `toHaveAttribute('current-value', ...)` instead of `toHaveValue()` (which only works on native `input`/`select`/`textarea`)

### Things to avoid

- Avoid "Nth element" selectors unless targeting table rows/cells
- Avoid `>> nth=N` and `>> text=MyText` with non-constant values; prefer `locator.nth(N)` or `locator.filter({ hasText: ... })`
- Avoid logic requiring `await` to narrow element selection (iterating, `getAttribute`) — use CSS selectors instead
- Avoid selecting inside Nimble shadow DOM when possible

## Authentication

### End2EndTests

Tests authenticate by adding session cookies to the browser context:

```typescript
test.beforeEach(async ({ context }) => {
    await Utils.addSessionCookies(context);
});
```

`Utils.addSessionCookies()`:
- Reads from config (session cookie or niuaId)
- Creates an API session key if needed
- Caches sessions by user ID for reuse across tests
- Adds cookies to the browser context

**Do NOT use `storageState` file-based auth** from the Playwright docs — this codebase uses cookie-based session management.

### Web acceptance tests

Acceptance tests run against a fake API server and typically don't need authentication.

## Fixtures and Test Context

### Extending the instrumented test

For tests needing shared state (setup/teardown), extend the instrumented `test`:

```typescript
import { test as baseTest } from '../shared/instrumentation/test';

interface AlarmsContext {
    alarmsToDelete: string[];
    alarmService: AlarmService;
}

const test = baseTest.extend<{ alarmsContext: AlarmsContext }>({
    alarmsContext: async ({}, use) => {
        const alarmsContext: AlarmsContext = {
            alarmsToDelete: [],
            alarmService: new AlarmService(),
        };
        await use(alarmsContext);
        // Cleanup runs even if test fails
    }
});
```

Key patterns:
- Define an interface for test context
- Extend `baseTest` (the instrumented test), not `@playwright/test`'s `test`
- Use the fixture's post-`use()` code for cleanup (no try/finally needed)

## Tagging and Filtering

### ProductAreaTags

Tests are tagged with product area tags for filtering:

```typescript
import { ProductAreaTags } from '../shared/product-area-tags';

test('alarm details', { tag: [ProductAreaTags.Alarms] }, async ({ page }) => {
    // ...
});
```

Tags are defined as an enum with `@` prefix values:
```typescript
export enum ProductAreaTags {
    Alarms = '@alarms',
    Assets = '@assets',
    Files = '@files',
    // etc.
}
```

### Environment-based test exclusion

Tests can be disabled per environment using `getDisabledTestsRegex()` from `config-utils.ts`, which builds regex patterns for `grepInvert` in the Playwright config.

## Linting Rules

Both End2EndTests and Web acceptance tests use `@ni/eslint-config-playwright` which extends standard Playwright ESLint rules.

### End2EndTests-specific rules

Key rules enforced via eslint:

```javascript
// REQUIRED: Use HTTP context wrapper instead of node-fetch directly
'no-restricted-imports': ['error', {
    paths: [{
        name: 'node-fetch',
        message: 'Use utils/http-context instead, so HTTP requests are logged and viewable in Playwright trace files.'
    }, {
        name: '@playwright/test',
        importNames: ['test'],
        message: "Don't import test from @playwright/test directly. Use shared/instrumentation/test."
    }]
}]

// REQUIRED: No test.only() in committed code
'no-restricted-syntax': ['error', {
    message: 'Focused tests (test.only()) should not be submitted.',
    selector: "CallExpression > MemberExpression[object.name='test'][property.name='only']"
}]

// REQUIRED: Return types on functions (improves readability)
'@typescript-eslint/explicit-function-return-type': ['error', { allowExpressions: true }]
```

### Web acceptance test rules

Some apps disable `playwright/prefer-web-first-assertions` where needed:
```javascript
rules: { 'playwright/prefer-web-first-assertions': 'off' }
```

## ESM Import Requirements

End2EndTests uses ES modules. This requires:

1. **File extensions on imports**: Include `.js` extension when importing `.ts` files:
   ```typescript
   import { Utils } from '../shared/utils/enterprise-utils.js';
   ```

2. **Separate `import type` for type-only imports**:
   ```typescript
   import type { Locator, Page } from '@playwright/test';
   import type { APIResponse } from '@playwright/test';
   ```
   Do not mix `import` with `import type` in the same statement.

## Assertions Best Practices

### Prefer auto-retrying assertions

```typescript
// GOOD - auto-retries until condition met
await expect(elementLocator).toContainText('ABC');
await expect(page).toHaveURL(/dashboard/);

// BAD - evaluates condition once (flaky)
await expect(aConstantValue).toEqual('ABC');
await expect(page.url()).toContain('dashboard');
```

### Use expect.poll() for custom conditions

When no built-in assertion works:
```typescript
await expect.poll(async () => {
    // async code returning some value
}).toEqual(expectedValue);
```

**Note**: Always chain an assertion method after `expect.poll()`. Forgetting it causes a vague "Property 'then' not found" error.

### Service/API-only assertions

For tests that only test HTTP endpoints (no browser), non-async assertions are fine:
```typescript
expect(response.body.abc).toEqual('def');
```
