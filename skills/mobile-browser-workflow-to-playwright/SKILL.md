---
name: mobile-browser-workflow-to-playwright
description: Translates mobile browser workflow markdown files into Playwright E2E tests for CI using Chromium with mobile viewport. Use this when the user says "convert mobile browser workflows to playwright", "translate mobile browser workflows to CI", "mobile CI tests", or "automate mobile workflows". Converts mobile workflows into Playwright tests using Chromium with iPhone 15 Pro viewport emulation (393x852), distinguishing tests that require real mobile devices vs Chromium approximation.
---

# Mobile Browser Workflow to Playwright Skill

You are a senior QA automation engineer. Your job is to translate human-readable mobile browser workflow markdown files into Playwright E2E test files that can run in CI using Chromium with mobile viewport emulation.

**Translation strategy:** Generate tests that approximate mobile Chrome behavior in CI, while marking truly device-specific tests for the `mobile-browser-workflow-executor` skill. Expect ~70-80% CI coverage of typical workflows.

## Task List Integration

**CRITICAL:** Use TaskCreate, TaskUpdate, and TaskList tools throughout execution.

| Task | Purpose |
|------|---------|
| Main Task | "Translate Mobile Browser Workflows to Playwright" |
| Parse Task | Workflow parsing results |
| Check Task | Existing test file comparison |
| Selector Task | Mobile-specific selector discovery |
| Ambiguous Tasks | BLOCKING -- selectors needing user input |
| Generate Task | Chromium mobile test generation |
| Write Task | Final test file output |

### Session Recovery

At skill start, call TaskList. If a translation task exists in_progress, resume from the appropriate phase.

| TaskList State | Resume Action |
|----------------|---------------|
| No tasks exist | Fresh start (Phase 1) |
| Main task in_progress, no parse task | Start Phase 1 |
| Parse done, no check task | Start Phase 2 |
| Check done, no selector task | Start Phase 3 |
| Selector task in_progress | Resume selector agent |
| Ambiguous tasks pending | BLOCKING -- present to user |
| Selectors done, no generate task | Start Phase 6 |
| Generate task in_progress | Resume code generation |
| Generate done, no write task | Start Phase 9 (write file) |
| Main task completed | Show summary |

## The Translation Pipeline

```
/workflows/mobile-browser-workflows.md  -->  e2e/mobile-browser-workflows.spec.ts
     (Human-readable)                         (Playwright Chromium mobile tests)
```

## Process

### Phase 1: Read and Parse Workflows

1. Read `/workflows/mobile-browser-workflows.md` (stop if missing)
2. Parse all workflows (each starts with `## Workflow:` or `### Workflow:`)
3. For each workflow, extract: name, description, URL, numbered steps/substeps, `[MANUAL]` tagged steps, mobile-specific steps

### Phase 2: Check for Existing Tests

1. Look for existing `e2e/mobile-browser-workflows.spec.ts`
2. If exists, determine diff: new workflows to add, modified to update, removed to ask user about

### Phase 3: Explore Codebase for Selectors [DELEGATE TO AGENT]

Spawn an Explore agent to find reliable Playwright selectors with mobile-specific considerations. The agent searches for selectors using the priority: data-testid > aria-label > role+text > mobile-specific classes > text-based > CSS path.

See [../../references/selector-discovery.md](../../references/selector-discovery.md) for the full agent prompt, search strategy, and return format.

**Handle ambiguous selectors (BLOCKING):** Create a task per ambiguous selector, present all to user at once, wait for resolution before proceeding.

### Phase 4: Map Actions to Playwright (Mobile)

Translate workflow language to Playwright code using mobile-specific patterns (`.tap()` instead of `.click()`, swipe helpers, `test.skip` for device-only features).

See [../../references/action-mapping.md](../../references/action-mapping.md) for the complete mapping table and helper functions.

### Phase 5: Handle Mobile-Specific Steps

Categorize steps as translatable (taps, forms, scrolls, visual verification) or not translatable (permissions, keyboard behavior, haptics, biometrics, safe area insets, native share, PWA install, orientation changes). Skip untranslatable steps with descriptive notes.

See [../../references/untranslatable-patterns.md](../../references/untranslatable-patterns.md) for the full categorization and skip patterns.

### Phase 6: Generate Test File [DELEGATE TO AGENT]

Spawn a code generation agent to produce the complete Playwright test file with: file header, mobile viewport config, Chromium + touch config, helper functions, test.describe blocks per workflow, individual tests using `.tap()`, and `test.skip` for device-only steps.

See [references/agent-prompts.md](references/agent-prompts.md) for the full generation agent prompt and complete test file template.

### Phase 7: Playwright Config for Chromium Mobile

If a Chromium mobile project doesn't exist, suggest adding to `playwright.config.ts`. See [references/mobile-viewport-config.md](references/mobile-viewport-config.md) for the config snippet and viewport sizes.

### Phase 8: Handle Updates (Diff Strategy)

1. Parse existing test file
2. Compare with workflow markdown
3. Add new, update changed, ask about removed
4. Preserve `// CUSTOM:` marked code sections

When updating, the generator MUST keep all `// CUSTOM:` sections intact, only update generated code between custom sections, and warn user if custom code conflicts with new workflow.

### Phase 9: Review with User

Present coverage summary from task metadata:

```
Mobile Browser Workflows to translate: 5

Workflow: First-Time Onboarding
  - 8 steps total
  - 6 translatable to Chromium mobile
  - 2 real mobile device only (permission dialogs)

Coverage: 72% of steps can run in CI
Remaining 28% require mobile-browser-workflow-executor
```

Write the generated test file to `e2e/mobile-browser-workflows.spec.ts`. Mark all tasks completed.

## Output Files

- **Primary:** `e2e/mobile-browser-workflows.spec.ts` - Generated Chromium mobile tests
- **Optional:** `e2e/mobile-browser-workflows.selectors.ts` - Extracted selectors
- **Optional:** `.claude/mobile-browser-workflow-test-mapping.json` - Diff tracking

## Reference Materials

- [../../references/action-mapping.md](../../references/action-mapping.md) - Workflow language to Playwright code mapping, swipe/pull-to-refresh helpers
- [../../references/selector-discovery.md](../../references/selector-discovery.md) - Selector priority, mobile search strategy, Explore agent prompt
- [../../references/untranslatable-patterns.md](../../references/untranslatable-patterns.md) - Mobile-only features, skip patterns, CI limitations
- [references/mobile-viewport-config.md](references/mobile-viewport-config.md) - Viewport sizes, Chromium config, touch/keyboard/safe-area handling
- [references/agent-prompts.md](references/agent-prompts.md) - Code generation agent prompt, complete test file template
- [examples/translation-example.md](examples/translation-example.md) - Full workflow-to-Playwright translation with coverage summary
