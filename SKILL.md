---
name: skyline-playwright-best-practices
description: Provides guidance for writing, debugging, and maintaining Playwright tests in TypeScript for the Skyline (SystemLink) codebase. Use when writing Playwright tests, fixing flaky tests, debugging failures, implementing Page Object Model, configuring CI/CD, optimizing performance, mocking APIs, handling authentication, testing accessibility (axe-core), file uploads/downloads, date/time mocking, geolocation, permissions, multi-tab/popup flows, error handling, multi-user collaboration, third-party services (OAuth/SSO), console error monitoring, global setup/teardown, test annotations (skip, fixme, slow), project dependencies, or iframes. Covers E2E smoke tests, user journey tests, and per-app acceptance tests. IMPORTANT: Always read skyline-conventions.md first for codebase-specific patterns that override generic Playwright guidance.
license: MIT
metadata:
  author: currents.dev (original), customized for Skyline/SystemLink
  version: "2.0"
---

# Playwright Best Practices (Skyline / SystemLink)

This skill provides guidance for Playwright test development in the Skyline (SystemLink) repository, covering End2EndTests (smoke + user journey) and Web/Workspaces acceptance tests.

**IMPORTANT**: Always consult [skyline-conventions.md](references/skyline-conventions.md) first. It documents codebase-specific patterns (instrumented test imports, custom element locators, page object conventions, authentication, tagging, linting rules) that take precedence over generic guidance.

## Activity-Based Reference Guide

Consult these references based on what you're doing:

### Skyline-Specific Patterns (Read First)

**When to use**: ANY Playwright work in this codebase

| Activity                                     | Reference Files                                                     |
| -------------------------------------------- | ------------------------------------------------------------------- |
| **Codebase conventions & required patterns** | [skyline-conventions.md](references/skyline-conventions.md)         |

### Writing New Tests

**When to use**: Creating new test files, writing test cases, implementing test scenarios

| Activity                           | Reference Files                                                                                                                                                                                        |
| ---------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Writing E2E tests**              | [skyline-conventions.md](references/skyline-conventions.md), [test-organization.md](references/test-organization.md), [locators.md](references/locators.md), [assertions-waiting.md](references/assertions-waiting.md) |
| **Writing API-only tests**         | [skyline-conventions.md](references/skyline-conventions.md), [test-organization.md](references/test-organization.md), [assertions-waiting.md](references/assertions-waiting.md)                         |
| **Structuring test code with POM** | [skyline-conventions.md](references/skyline-conventions.md), [page-object-model.md](references/page-object-model.md), [test-organization.md](references/test-organization.md)                           |
| **Setting up test data/fixtures**  | [fixtures-hooks.md](references/fixtures-hooks.md), [test-data.md](references/test-data.md)                                                                                                             |
| **Handling authentication**        | [skyline-conventions.md](references/skyline-conventions.md), [fixtures-hooks.md](references/fixtures-hooks.md)                                                                                          |
| **Testing date/time features**     | [clock-mocking.md](references/clock-mocking.md)                                                                                                                                                        |
| **Testing file upload/download**   | [file-operations.md](references/file-operations.md)                                                                                                                                                     |
| **Testing accessibility**          | [accessibility.md](references/accessibility.md)                                                                                                                                                         |
| **Using test annotations**         | [annotations.md](references/annotations.md)                                                                                                                                                             |
| **Testing iframes**                | [iframes.md](references/iframes.md)                                                                                                                                                                     |

### Browser APIs & Multi-Context

**When to use**: Testing geolocation, permissions, multi-tab flows, OAuth

| Activity                   | Reference Files                                                                              |
| -------------------------- | -------------------------------------------------------------------------------------------- |
| **Geolocation mocking**   | [browser-apis.md](references/browser-apis.md)                                                |
| **Permission handling**   | [browser-apis.md](references/browser-apis.md)                                                |
| **Clipboard testing**     | [browser-apis.md](references/browser-apis.md)                                                |
| **Multi-tab/popup flows** | [multi-context.md](references/multi-context.md)                                              |
| **OAuth popup handling**  | [third-party.md](references/third-party.md), [multi-context.md](references/multi-context.md) |

### Debugging & Troubleshooting

**When to use**: Test failures, element not found, timeouts, unexpected behavior

| Activity                                          | Reference Files                                                                                                                                 |
| ------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| **Debugging test failures**                       | [debugging.md](references/debugging.md), [assertions-waiting.md](references/assertions-waiting.md)                                              |
| **Fixing flaky tests**                            | [flaky-tests.md](references/flaky-tests.md), [debugging.md](references/debugging.md), [assertions-waiting.md](references/assertions-waiting.md) |
| **Debugging flaky parallel runs**                 | [flaky-tests.md](references/flaky-tests.md), [performance.md](references/performance.md), [fixtures-hooks.md](references/fixtures-hooks.md)     |
| **Ensuring test isolation / avoiding state leak** | [flaky-tests.md](references/flaky-tests.md), [fixtures-hooks.md](references/fixtures-hooks.md), [performance.md](references/performance.md)     |
| **Fixing selector issues**                        | [skyline-conventions.md](references/skyline-conventions.md), [locators.md](references/locators.md), [debugging.md](references/debugging.md)      |
| **Investigating timeout issues**                  | [assertions-waiting.md](references/assertions-waiting.md), [debugging.md](references/debugging.md)                                              |
| **Using trace viewer**                            | [debugging.md](references/debugging.md)                                                                                                         |
| **Debugging race conditions**                     | [flaky-tests.md](references/flaky-tests.md), [debugging.md](references/debugging.md), [assertions-waiting.md](references/assertions-waiting.md) |
| **Debugging console/JS errors**                   | [console-errors.md](references/console-errors.md), [debugging.md](references/debugging.md)                                                      |

### Error & Edge Case Testing

**When to use**: Testing error states, network failures, validation

| Activity                       | Reference Files                                                                                        |
| ------------------------------ | ------------------------------------------------------------------------------------------------------ |
| **Error boundary testing**     | [error-testing.md](references/error-testing.md)                                                        |
| **Network failure simulation** | [error-testing.md](references/error-testing.md), [network-advanced.md](references/network-advanced.md) |
| **Loading state testing**      | [error-testing.md](references/error-testing.md)                                                        |
| **Form validation testing**    | [error-testing.md](references/error-testing.md)                                                        |

### Multi-User & Collaboration Testing

**When to use**: Testing features involving multiple users, roles, or concurrent access

| Activity                       | Reference Files                                     |
| ------------------------------ | --------------------------------------------------- |
| **Multiple users in one test** | [multi-user.md](references/multi-user.md)           |
| **Role-based access testing**  | [multi-user.md](references/multi-user.md)           |
| **Concurrent action testing**  | [multi-user.md](references/multi-user.md)           |

### Refactoring & Maintenance

**When to use**: Improving existing tests, code review, reducing duplication

| Activity                              | Reference Files                                                                                                                                                        |
| ------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Refactoring to Page Object Model**  | [skyline-conventions.md](references/skyline-conventions.md), [page-object-model.md](references/page-object-model.md), [test-organization.md](references/test-organization.md) |
| **Improving test organization**       | [test-organization.md](references/test-organization.md), [page-object-model.md](references/page-object-model.md)                                                       |
| **Extracting common setup/teardown**  | [fixtures-hooks.md](references/fixtures-hooks.md)                                                                                                                       |
| **Replacing brittle selectors**       | [skyline-conventions.md](references/skyline-conventions.md), [locators.md](references/locators.md)                                                                      |
| **Removing explicit waits**           | [assertions-waiting.md](references/assertions-waiting.md)                                                                                                               |
| **Creating test data factories**      | [test-data.md](references/test-data.md)                                                                                                                                 |

### Infrastructure & Configuration

**When to use**: Setting up projects, configuring CI/CD, optimizing performance

| Activity                                | Reference Files                                                                                                                                                       |
| --------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Configuring Playwright project**      | [skyline-conventions.md](references/skyline-conventions.md), [test-organization.md](references/test-organization.md), [projects-dependencies.md](references/projects-dependencies.md) |
| **Setting up CI/CD pipelines**          | [skyline-conventions.md](references/skyline-conventions.md), [performance.md](references/performance.md)                                                                               |
| **Global setup & teardown**             | [global-setup.md](references/global-setup.md)                                                                                                                         |
| **Project dependencies**                | [projects-dependencies.md](references/projects-dependencies.md)                                                                                                       |
| **Optimizing test performance**         | [performance.md](references/performance.md), [test-organization.md](references/test-organization.md)                                                                  |
| **Configuring parallel execution**      | [performance.md](references/performance.md)                                                                                                                            |
| **Isolating test data between workers** | [fixtures-hooks.md](references/fixtures-hooks.md), [performance.md](references/performance.md)                                                                        |

### Advanced Patterns

**When to use**: Complex scenarios, API mocking, network interception

| Activity                           | Reference Files                                                                                                  |
| ---------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| **Mocking API responses**          | [test-organization.md](references/test-organization.md), [network-advanced.md](references/network-advanced.md)   |
| **Network interception**           | [network-advanced.md](references/network-advanced.md), [assertions-waiting.md](references/assertions-waiting.md) |
| **GraphQL mocking**                | [network-advanced.md](references/network-advanced.md)                                                            |
| **HAR recording/playback**         | [network-advanced.md](references/network-advanced.md)                                                            |
| **Custom fixtures**                | [fixtures-hooks.md](references/fixtures-hooks.md)                                                                |
| **Advanced waiting strategies**    | [assertions-waiting.md](references/assertions-waiting.md)                                                        |
| **OAuth/SSO handling**             | [third-party.md](references/third-party.md), [multi-context.md](references/multi-context.md)                     |
| **Failing on console errors**      | [console-errors.md](references/console-errors.md)                                                                |
| **Test annotations (skip, fixme)** | [annotations.md](references/annotations.md)                                                                      |
| **Test steps for reporting**       | [annotations.md](references/annotations.md)                                                                      |

## Quick Decision Tree

```
What are you doing?
│
├─ FIRST: Read skyline-conventions.md for codebase-specific patterns
│
├─ Writing a new test?
│  ├─ E2E test → skyline-conventions.md, test-organization.md, locators.md, assertions-waiting.md
│  ├─ API-only test → skyline-conventions.md, test-organization.md, assertions-waiting.md
│  ├─ Accessibility test → accessibility.md
│  └─ Multi-user test → multi-user.md
│
├─ Testing specific features?
│  ├─ File upload/download → file-operations.md
│  ├─ Date/time dependent → clock-mocking.md
│  ├─ Geolocation/permissions → browser-apis.md
│  ├─ OAuth/SSO handling → third-party.md, multi-context.md
│  └─ iFrames → iframes.md
│
├─ Test is failing/flaky?
│  ├─ Flaky test investigation → flaky-tests.md
│  ├─ Element not found → skyline-conventions.md, locators.md, debugging.md
│  ├─ Timeout issues → assertions-waiting.md, debugging.md
│  ├─ Race conditions → flaky-tests.md, debugging.md
│  ├─ Flaky only with multiple workers → flaky-tests.md, performance.md
│  ├─ State leak / isolation → flaky-tests.md, fixtures-hooks.md
│  ├─ Console/JS errors → console-errors.md, debugging.md
│  └─ General debugging → debugging.md
│
├─ Testing error scenarios?
│  ├─ Network failures → error-testing.md, network-advanced.md
│  ├─ Error boundaries → error-testing.md
│  └─ Form validation → error-testing.md
│
├─ Refactoring existing code?
│  ├─ Implementing POM → skyline-conventions.md, page-object-model.md
│  ├─ Improving selectors → skyline-conventions.md, locators.md
│  ├─ Extracting fixtures → fixtures-hooks.md
│  └─ Creating data factories → test-data.md
│
├─ Setting up infrastructure?
│  ├─ CI/CD → skyline-conventions.md, performance.md
│  ├─ Global setup/teardown → global-setup.md
│  ├─ Project dependencies → projects-dependencies.md
│  ├─ Test performance → performance.md
│  └─ Project config → skyline-conventions.md, test-organization.md, projects-dependencies.md
│
└─ Organizing tests?
   ├─ Skip/fixme/slow tests → annotations.md
   ├─ Test steps → annotations.md
   └─ Conditional execution → annotations.md
```

## Test Validation Loop

After writing or modifying tests:

### End2EndTests

1. **Run smoke tests**: `npm run test:smoke` (from `End2EndTests/`)
2. **Run user journey tests**: `npm run test:userjourney`
3. **If tests fail**:
   - Review error output and trace (`npx playwright show-trace`)
   - Fix locators, waits, or assertions
   - Re-run tests
4. **Run lint**: `npm run lint`

### Web acceptance tests

1. **Start the fake API server** (per-app instructions)
2. **Run tests**: `npx playwright test` (from `<App>/acceptance-tests/`)
3. **If tests fail**: Review trace and fix
