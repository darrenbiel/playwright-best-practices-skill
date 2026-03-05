```

░█▀█░█░░░█▀█░█░█░█░█░█▀▄░▀█▀░█▀▀░█░█░▀█▀░░░█▀▄░█▀▀░█▀▀░▀█▀░░░█▀█░█▀▄░█▀█░█▀▀░▀█▀░▀█▀░█▀▀░█▀▀░█▀▀░
░█▀▀░█░░░█▀█░░█░░█▄█░█▀▄░░█░░█░█░█▀█░░█░░░░█▀▄░█▀▀░▀▀█░░█░░░░█▀▀░█▀▄░█▀█░█░░░░█░░░█░░█░░░█▀▀░▀▀█░
░▀░░░▀▀▀░▀░▀░░▀░░▀░▀░▀░▀░▀▀▀░▀▀▀░▀░▀░░▀░░░░▀▀░░▀▀▀░▀▀▀░░▀░░░░▀░░░▀░▀░▀░▀░▀▀▀░░▀░░▀▀▀░▀▀▀░▀▀▀░▀▀▀░
```

# Playwright Best Practices Skill (Skyline / SystemLink)

A customized fork of the [currents.dev Playwright Best Practices Skill](https://github.com/currents-dev/playwright-best-practices-skill), tailored for the **Skyline (SystemLink)** codebase.

This skill gives the AI specialized guidance for writing, debugging, and maintaining Playwright tests in TypeScript, with codebase-specific patterns for:

- End2EndTests (smoke + user journey projects)
- Web/Workspaces acceptance tests (17 apps with fake API servers)
- Custom web components (`nimble-*`, `sl-*`)
- Azure DevOps CI pipelines
- Cookie-based authentication
- OpenTelemetry-instrumented test imports

## Skyline Customizations

See [CHANGES.md](CHANGES.md) for a detailed description of all modifications made to the original skill.

Key changes:
- **Added** `skyline-conventions.md` — central reference for codebase-specific patterns
- **Removed** 12 irrelevant reference files (mobile, electron, extensions, canvas, service workers, component testing, performance testing, security testing, coverage, websockets, i18n, ci-cd)
- **Modified** 6 reference files with Skyline-specific notes and examples
- **Rewrote** `SKILL.md` with updated activity tables and decision tree

## Integration with Skyline

The [fork repo](https://github.com/darrenbiel/playwright-best-practices-skill) is embedded in the Skyline monorepo as a **git subtree** at:

```
.github/skills/skyline-playwright-best-practices/
```

The fork's `skyline-customization` branch is the source of truth.

## Contributing

### Source of truth

The [fork repo](https://github.com/darrenbiel/playwright-best-practices-skill) (branch `skyline-customization`) is the source of truth. Make edits there, then sync to Skyline.

### Making changes

1. Clone the fork and check out the `skyline-customization` branch:
   ```bash
   git clone https://github.com/darrenbiel/playwright-best-practices-skill.git
   cd playwright-best-practices-skill
   git checkout skyline-customization
   ```

2. Make your edits (add/modify reference files, update SKILL.md, etc.)

3. Commit with a descriptive message and push:
   ```bash
   git add -A
   git commit -m "Update locators.md with new nimble component patterns"
   git push origin skyline-customization
   ```

4. Sync to Skyline by running this from the Skyline repo root:
   ```bash
   git subtree pull --prefix=.github/skills/skyline-playwright-best-practices \
     https://github.com/darrenbiel/playwright-best-practices-skill.git \
     skyline-customization --squash
   ```
   This creates a merge commit in Skyline with the latest fork content.

### Quick fixes directly in Skyline

If you need to make a quick fix directly in the Skyline monorepo, edit the files under `.github/skills/skyline-playwright-best-practices/` and commit normally. To push those changes back to the fork:

```bash
git subtree push --prefix=.github/skills/skyline-playwright-best-practices \
  https://github.com/darrenbiel/playwright-best-practices-skill.git \
  skyline-customization
```

> **Note:** `subtree push` walks the full Skyline history to extract changes, which can be slow. Prefer making changes in the fork when possible.

### Important guidelines

- Always update `skyline-conventions.md` when codebase patterns change (e.g., new custom elements, auth changes, new linting rules)
- When removing a reference file, search for and update all cross-references in other files and SKILL.md
- Update CHANGES.md when making significant modifications

## What's Inside

### Skyline-Specific

| Topic                | Reference                | Use for                                             |
| -------------------- | ------------------------ | --------------------------------------------------- |
| Codebase conventions | `skyline-conventions.md` | **Read first.** Imports, locators, auth, config, tags |

### Core Testing

| Topic                | Reference               | Use for                                          |
| -------------------- | ----------------------- | ------------------------------------------------ |
| Debugging            | `debugging.md`          | Trace viewer, inspector, common issues           |
| Flaky tests          | `flaky-tests.md`        | Detection, diagnosis, fixing, quarantine         |
| Test organization    | `test-organization.md`  | Structure, config, E2E/API tests                 |
| Locators             | `locators.md`           | Selectors, custom elements, shadow DOM           |
| Assertions & waiting | `assertions-waiting.md` | Expect APIs, auto-waiting, polling               |
| Page Object Model    | `page-object-model.md`  | POM structure, composition patterns              |
| Fixtures & hooks     | `fixtures-hooks.md`     | Setup, teardown, auth, custom fixtures           |
| Test data            | `test-data.md`          | Factories, Faker, data-driven testing            |
| Annotations          | `annotations.md`        | skip, fixme, slow, test steps, ProductAreaTags   |

### Specialized Testing

| Topic            | Reference            | Use for                                        |
| ---------------- | -------------------- | ---------------------------------------------- |
| Accessibility    | `accessibility.md`   | Axe-core, keyboard nav, ARIA, focus management |
| File operations  | `file-operations.md` | Upload, download, drag-and-drop                |
| Clock mocking    | `clock-mocking.md`   | Date/time mocking, timezones, timers           |
| Browser APIs     | `browser-apis.md`    | Geolocation, permissions, clipboard            |
| Multi-context    | `multi-context.md`   | Popups, new tabs, OAuth flows                  |
| Multi-user       | `multi-user.md`      | Collaboration, RBAC, concurrent actions        |
| iFrames          | `iframes.md`         | Cross-origin, nested, dynamic iframes          |
| Error testing    | `error-testing.md`   | Error boundaries, offline, network failures    |

### Infrastructure & Advanced

| Topic            | Reference                  | Use for                                 |
| ---------------- | -------------------------- | --------------------------------------- |
| Performance      | `performance.md`           | Parallel runs, sharding, optimization   |
| Global setup     | `global-setup.md`          | globalSetup/Teardown, DB migrations     |
| Projects         | `projects-dependencies.md` | Project config, dependencies, filtering |
| Network advanced | `network-advanced.md`      | GraphQL, HAR, request modification      |
| Third-party      | `third-party.md`           | OAuth, SSO mocking                      |
| Console errors   | `console-errors.md`        | Capturing and failing on JS errors      |

## License

MIT — Original skill by [currents.dev](https://currents.dev), customized for Skyline/SystemLink.
