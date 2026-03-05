# Skyline Customization Changes

This document describes all modifications made to the original `playwright-best-practices` skill (from [currents.dev](https://currents.dev)) to align it with the Skyline (SystemLink) codebase implementation.

## Rationale

The original skill provides comprehensive Playwright guidance covering many platforms, frameworks, and testing types. The Skyline codebase uses a specific subset of Playwright features (Chromium-only, TypeScript/ESM, Angular web apps with custom elements, Azure DevOps CI). This customization removes irrelevant guidance and adds codebase-specific patterns discovered by surveying:

- `End2EndTests/` — E2E smoke and user journey tests
- `Web/Workspaces/*/acceptance-tests/` — Per-app acceptance tests (17 apps)
- `@ni/eslint-config-playwright` — Internal linting rules

---

## Commit 1: Delete 11 Irrelevant Reference Files

**Files removed:**

| File | Reason |
|------|--------|
| `mobile-testing.md` | Skyline is Chromium-only (Desktop Chrome). No mobile device emulation, touch gestures, or responsive viewport testing. |
| `electron.md` | Skyline is a web application, not an Electron app. |
| `browser-extensions.md` | No browser extension testing in the codebase. |
| `canvas-webgl.md` | No canvas or WebGL testing. Skyline uses standard DOM/custom elements. |
| `service-workers.md` | No service workers or PWA features. Skyline apps are server-rendered SPA. |
| `component-testing.md` | Skyline does not use Playwright Component Testing (`@playwright/experimental-ct-*`). Component testing is done with other tools. |
| `performance-testing.md` | No Lighthouse/Web Vitals integration in Playwright tests. Performance is measured separately. |
| `security-testing.md` | No XSS/CSRF/auth security testing via Playwright. Security is handled by separate tooling (Snyk, etc.). |
| `test-coverage.md` | No Istanbul/V8 code coverage instrumentation in Playwright tests. |
| `websockets.md` | No WebSocket testing patterns in the codebase. |
| `i18n.md` | No internationalization/localization testing via Playwright. |

## Commit 2: Create skyline-conventions.md

**New file**: `references/skyline-conventions.md` (~400 lines)

Central reference documenting all codebase-specific patterns that override generic Playwright guidance:

- **Repository layout**: End2EndTests vs Web/Workspaces acceptance-tests structure
- **Test import requirement**: Must use instrumented `test` from `tests/shared/instrumentation/test.js` (OpenTelemetry), enforced by eslint
- **File naming**: `*.spec.ts` for tests, `*.page.ts` for page objects
- **Config patterns**: No `fullyParallel`, no `webServer`, Chromium-only projects
- **Page objects**: Composition over inheritance, no `BasePage` abstract class
- **Custom element locators**: Priority order for `nimble-*` and `sl-*` web components
- **Authentication**: Cookie-based via `Utils.addSessionCookies(context)`, not `storageState`
- **Fixtures & context**: `test.use({...})` per describe block for shared config
- **Tagging**: `ProductAreaTags` enum for categorizing End2EndTests
- **Linting**: `@ni/eslint-config-playwright` rules including forced instrumented import
- **ESM imports**: Must use `.js` extensions and `import type` separation
- **Assertion best practices**: `toPass()` for retry, `expect.soft()` for non-blocking

## Commit 3: Rewrite SKILL.md

**Complete rewrite** of the skill entry point:

- Updated frontmatter: renamed to `skyline-playwright-best-practices`, version 2.0
- Added `skyline-conventions.md` as primary "Read First" reference throughout
- Removed all references to deleted files from activity tables and decision tree
- Removed "Mobile & Responsive" section entirely
- Removed rows: component testing, visual regression, canvas/WebGL, security testing, test coverage, i18n, Electron, browser extensions, payment/SMS gateway, WebSocket
- Renamed "Real-Time & Browser APIs" to "Browser APIs & Multi-Context"
- Updated decision tree to only reference remaining files
- Updated test validation loop with actual npm scripts (`npm run test:smoke`, `npm run test:userjourney`)
- Added separate End2EndTests and Web acceptance test validation sections

## Commit 4: Update test-organization.md

- Added Skyline-specific note at top referencing `skyline-conventions.md`
- Removed "Component Tests" section (component-testing.md was deleted)
- Removed "Visual Regression Tests" section (canvas-webgl.md was deleted)
- Removed `fullyParallel: true` and `webServer` from example config
- Added note about Skyline config patterns
- Updated directory structure to remove component/ and visual/ dirs
- Added `ProductAreaTags` subsection for Skyline test categorization
- Updated Related References

## Commit 5: Update page-object-model.md

- Added Skyline note: composition over inheritance, no `BasePage` pattern
- Added caveat to `BasePage` section marking it as generic reference only
- Added `skyline-conventions.md` to Related References

## Commit 6: Update locators.md

- Added Skyline note about `nimble-*`/`sl-*` custom web components
- Renamed "Shadow DOM" section to "Shadow DOM & Custom Elements"
- Added "Skyline Custom Elements" subsection with practical examples
- Showed preferred locator patterns for nimble-text-field, nimble-table-row, sl-drawer
- Added `skyline-conventions.md` to Related References

## Commit 7: Update ci-cd.md

- Added top-level note: Skyline uses Azure DevOps, not GitHub Actions
- Renamed "GitHub Actions" section to "GitHub Actions (General Reference)"
- Added note that GH Actions examples are kept as generic reference
- Removed `github` reporter from CI configuration examples
- Removed `fullyParallel: true` from CI config reference
- Added Skyline notes about custom reporter and actual CI patterns

## Commit 8: Update fixtures-hooks.md

- Added top-level note: must use instrumented test import
- Added note about cookie-based auth via `Utils.addSessionCookies(context)`
- Added caveat to `storageState` section (not used in Skyline)
- Added `skyline-conventions.md` to Related References

## Commit 9: Update annotations.md

- Added top-level note about `ProductAreaTags` enum usage
- Replaced mobile/desktop conditional annotation examples with environment-based example
- Added note that browser/mobile conditionals are not applicable (Chromium-only)

---

## Files Kept As-Is (24 reference files)

These files contain generic Playwright guidance that is applicable and does not conflict with Skyline patterns:

| File | Content |
|------|---------|
| `accessibility.md` | axe-core integration, WCAG testing |
| `assertions-waiting.md` | Auto-waiting, assertions, `toPass()` |
| `browser-apis.md` | Geolocation, permissions, clipboard |
| `clock-mocking.md` | Date/time mocking |
| `console-errors.md` | Console error monitoring |
| `debugging.md` | Trace viewer, Inspector, debugging |
| `error-testing.md` | Error boundaries, network failures |
| `file-operations.md` | Upload/download testing |
| `flaky-tests.md` | Flaky test investigation/fixing |
| `global-setup.md` | Global setup/teardown patterns |
| `iframes.md` | iframe testing |
| `multi-context.md` | Multi-tab/popup flows |
| `multi-user.md` | Multi-user testing |
| `network-advanced.md` | API mocking, GraphQL, HAR |
| `performance.md` | Parallel execution, sharding |
| `projects-dependencies.md` | Project configuration |
| `test-data.md` | Test data factories, Faker |
| `third-party.md` | OAuth/SSO handling |
