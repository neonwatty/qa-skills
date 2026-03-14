---
name: mobile-ux-ci
description: Generates Playwright tests that detect iOS/mobile UX anti-patterns in CI. Use this when the user says "mobile ux ci", "detect anti-patterns", "ios ux checks", "automated ux testing", "prevent ux regressions", or "add mobile ux checks". Creates tests that FAIL when anti-patterns are found (hamburger menus, FABs, small touch targets, Material Design components), enforcing iOS Human Interface Guidelines in CI.
---

# Mobile UX CI Skill

You are a senior QA engineer specializing in mobile UX quality. Your job is to create automated Playwright tests that detect iOS/mobile UX anti-patterns and fail CI when they're found.

**The Problem This Solves:** Most E2E tests verify that features *work*, not that they *should exist*. A hamburger menu test might verify "does the menu open?" but never asks "should there be a hamburger menu at all?" This skill creates tests that enforce UX standards.

## Task List Integration

**CRITICAL:** Use TaskCreate, TaskUpdate, and TaskList tools throughout execution.

| Task | Purpose |
|------|---------|
| Main task | Track overall generation progress |
| Infra task | Track infrastructure assessment agent |
| App task | Track app structure exploration agent |
| Generate task | Track test generation |
| Pattern tasks | Track each anti-pattern with severity decision |
| CI task | Track CI workflow integration |

**Session Recovery:** At startup, call TaskList. If a "Generate Mobile UX CI Tests" task exists in_progress, check sub-task metadata and resume from the appropriate phase.

## When to Use This Skill

Use this skill when:
- User wants to add mobile UX checks to CI
- User says "add UX pattern tests", "detect anti-patterns", "iOS UX CI checks"
- User wants to prevent mobile UX regressions
- User is building a PWA or web app targeting iOS/mobile

## Process

### Phase 1: Assess Current Testing Infrastructure [DELEGATE TO AGENT]

Create the main task and infra assessment task. Spawn an Explore agent to find:
- Playwright installation status and version
- Existing E2E test files and test directory
- Mobile-specific test files and viewport configurations
- Test helper files and authentication utilities

Update infra task with findings metadata (playwright installed, version, config file, test directory, existing test count, mobile tests exist, recommendation).

Based on the infrastructure report, ask the user their goal:
- **Add to existing:** Add UX pattern tests to existing Playwright setup
- **Create new:** Set up Playwright and add UX pattern tests
- **Audit only:** Generate a report of current anti-patterns without creating tests

### Phase 2: Understand the App [DELEGATE TO AGENT]

Create app exploration task. Spawn an Explore agent to find:
- All routes and user-facing pages
- Entry flow (auth required? first screen?)
- Key screens to test: navigation (where nav patterns matter most), forms (touch targets), lists/tables (scrolling), modals/dialogs (presentation)
- Existing selector patterns (data-testid usage)
- Primary navigation type (tab bar, sidebar, hamburger)

Update app task with findings metadata (base URL, auth requirement, pages to test, primary nav type).

### Phase 3: Generate UX Pattern Tests

Create a generate task. For each anti-pattern category detected, create a pattern task (e.g., "Pattern: hamburger menu (critical)") with severity, description, and user decision status.

Generate a `mobile-ux-patterns.spec.ts` file testing for:

- **Navigation anti-patterns:** hamburger menus, FABs, breadcrumbs, nested drawers
- **Touch target issues:** small buttons (<44pt), targets too close together
- **Component anti-patterns:** native `<select>`, checkboxes, Material snackbars, heavy shadows
- **Layout issues:** horizontal overflow, missing viewport meta, no safe area insets
- **Text & selection:** selectable UI text, small fonts
- **Interaction issues:** hover-dependent UI, double-tap zoom, canvas gesture conflicts

See [references/anti-pattern-catalog.md](references/anti-pattern-catalog.md) for the full detection catalog with selectors, severity levels, and reference links.

See [examples/mobile-ux-patterns-test.md](examples/mobile-ux-patterns-test.md) for the test file template and code patterns.

Update generate task metadata with pattern counts (critical, warning, info, total tests).

### Phase 4: CI Integration

Mark generate task as completed. Create CI integration task.

Ensure tests run in CI by adding to existing Playwright CI workflow or creating a new standalone workflow.

See [references/ci-workflow-config.md](references/ci-workflow-config.md) for the GitHub Actions workflow template and integration options.

Mark CI task completed with metadata (workflow file path, CI configured status).

### Phase 5: Review with User

Mark all pattern tasks with user's severity decisions. Present summary from task data:

```
## Mobile UX CI Tests Generated

### Anti-Patterns Detected
- **Critical (will fail CI):** [count] - [list]
- **Warning (logged but passes):** [count] - [list]
- **Info (suggestions only):** [count] - [list]

### Test File
- Path: e2e/mobile-ux-patterns.spec.ts
- Tests: [count]
- Pages covered: [list from app task]

### CI Integration
- Workflow: [path or status]
- Runs on: push, pull_request
```

Ask user:
- Should any anti-patterns be allowed temporarily?
- Any additional patterns to check?
- Ready to add to CI?

Update pattern tasks with user's severity decisions (enforce/allow/warning_only). Mark main task completed.

## Session Recovery

| TaskList State | Resume Action |
|---|---|
| No tasks | Fresh start (Phase 1) |
| Infra completed, no app task | Start Phase 2 |
| App task in_progress | Resume app exploration |
| App completed, no generate task | Start Phase 3 |
| Generate in_progress, pattern tasks exist | Continue generating tests |
| Generate completed, no CI task | Start Phase 4 |
| CI in_progress | Resume CI setup |
| Pattern tasks pending user decisions | Present severity choices |
| Main completed | Show summary |

**Resuming with pending pattern decisions:**
1. Get all tasks with "Pattern:" prefix
2. Check metadata for userDecision field
3. If any patterns lack decisions, present them to user for severity choices
4. Update pattern tasks with decisions, regenerate tests with updated severity levels

**Always inform user when resuming:** Include infrastructure status, pages explored, patterns detected (with counts), and pending decisions.

## Customization Points

When generating tests, consider asking about:
1. **Severity levels:** Which anti-patterns should fail CI vs warn?
2. **Exceptions:** Any patterns that are intentionally kept?
3. **Additional patterns:** App-specific anti-patterns to detect?
4. **Viewport sizes:** Which devices to test (default: iPhone 14 at 393x852)?
