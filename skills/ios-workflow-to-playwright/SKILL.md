---
name: ios-workflow-to-playwright
description: Translates iOS workflow markdown files into Playwright E2E tests for CI using WebKit with mobile viewport. Use this when the user says "convert ios workflows to playwright", "translate ios workflows to CI", "generate webkit mobile tests", or "automate ios workflows". Converts iOS workflows into Playwright tests with iPhone viewport emulation, marking tests that require real iOS Simulator vs WebKit approximation.
---

# iOS Workflow to Playwright Skill

You are a senior QA automation engineer translating human-readable iOS workflow markdown files into Playwright E2E test files that run in CI using WebKit with mobile viewport emulation.

## The Translation Pipeline

```
/workflows/ios-workflows.md     ->     e2e/ios-mobile-workflows.spec.ts
     (Human-readable)                    (Playwright WebKit mobile tests)
```

See [references/webkit-vs-ios.md](references/webkit-vs-ios.md) for what WebKit can/cannot approximate versus real iOS Simulator.

## When to Use This Skill

Use when the user has refined iOS workflows via `ios-workflow-executor` and wants to promote them to CI, or says "convert ios workflows to CI" / "generate mobile tests."

## Task List Integration

**CRITICAL:** Use TaskCreate, TaskUpdate, and TaskList tools throughout for progress tracking and session recovery.

### Task Hierarchy

| Task | Purpose |
|------|---------|
| `Translate iOS Workflows to Playwright` | Main task |
| `Parse: [N] workflows from ios-workflows.md` | Workflow parsing |
| `Check: existing ios-mobile-workflows.spec.ts` | Existing file check |
| `Selectors: finding mobile-specific selectors` | Agent: selector discovery |
| `Ambiguous: Step [N.M] - [description]` | BLOCKING: needs user input |
| `Generate: WebKit mobile tests` | Agent: code generation |
| `Write: e2e/ios-mobile-workflows.spec.ts` | Final file write |

### Session Recovery

At skill start, call `TaskList`. If a main task exists with `in_progress`, check child task states and resume.

| Task State | Resume Action |
|-----------|---------------|
| No tasks exist | Fresh start (Phase 1) |
| Main in_progress, no parse task | Start Phase 1 |
| Parse complete, no check task | Start Phase 2 |
| Check complete, no selector task | Start Phase 3 |
| Selector task in_progress | Resume selector agent |
| Ambiguous tasks pending | BLOCKING: present to user |
| Selector complete, no generate task | Start Phase 6 |
| Generate in_progress | Resume code generation agent |
| Generate complete, no write task | Start Phase 9 (write file) |
| Main completed | Show summary |

Always inform user: "Resuming: Source [file], Target [file], Workflows [count], Current state [description], Pending [blocking tasks]"

## Process

### Phase 1: Read and Parse Workflows

Create main task `"Translate iOS Workflows to Playwright"` and mark `in_progress`.

1. Read `/workflows/ios-workflows.md` (stop if missing)
2. Parse all workflows (each starts with `## Workflow:` or `### Workflow:`)
3. For each workflow extract: name, description, URL, numbered steps/substeps, `[MANUAL]` steps, iOS-specific steps

Create parse task with metadata: `workflowCount`, `totalSteps`, `iosSpecificSteps`, `manualSteps`.

### Phase 2: Check for Existing Tests

Look for existing `e2e/ios-mobile-workflows.spec.ts`. If it exists, determine diff: new workflows to add, modified workflows to update, removed workflows to ask about.

### Phase 3: Explore Codebase for Selectors [DELEGATE TO AGENT]

Spawn an Explore agent (sonnet) to find reliable Playwright selectors for each workflow step, with mobile-specific considerations.

See [../../references/selector-discovery.md](../../references/selector-discovery.md) for selector priority, search strategy, and return format.
See [references/agent-prompts.md](references/agent-prompts.md) for the full selector discovery agent prompt.

**Handle ambiguous selectors (BLOCKING):** For each ambiguous selector, create a pending task. Present all to user at once. Wait for ALL to be resolved before proceeding.

### Phase 4: Map Actions to Playwright (Mobile)

See [../../references/action-mapping.md](../../references/action-mapping.md) for the complete workflow-language-to-Playwright mapping table.

Key patterns: `Tap` -> `.tap()`, `Long press` -> `.click({ delay: 500 })`, `Swipe` -> custom helper, `Pinch` -> `test.skip()`, `[MANUAL]` -> `test.skip()`.

See [examples/webkit-helpers.md](examples/webkit-helpers.md) for the swipe helper and pull-to-refresh helper code.

### Phase 5: Handle iOS-Specific Steps

See [../../references/untranslatable-patterns.md](../../references/untranslatable-patterns.md) for the full list of translatable vs non-translatable patterns and iOS-only features.

Non-translatable steps get `test.skip()` with a note pointing to `ios-workflow-executor`.

### Phase 6: Generate Test File [DELEGATE TO AGENT]

Spawn a general-purpose agent (sonnet) to generate the complete Playwright test file with: file header, mobile viewport config (iPhone 14: 393x852), WebKit + touch config via `test.use()`, helper functions, `test.describe` blocks per workflow, `.tap()` for touch interactions, `test.skip` for iOS-only steps.

See [references/agent-prompts.md](references/agent-prompts.md) for the full code generation agent prompt.
See [examples/translation-example.md](examples/translation-example.md) for a complete workflow-to-test translation example.

### Phase 7: Playwright Config for WebKit Mobile

If no WebKit mobile project exists, suggest adding to `playwright.config.ts`:

```typescript
{
  name: 'Mobile Safari',
  use: {
    ...devices['iPhone 14'],
    browserName: 'webkit',
  },
},
```

### Phase 8: Handle Updates (Diff Strategy)

Parse existing test file, compare with workflow markdown. Add new, update changed, ask about removed. Preserve `// CUSTOM:` marked code.

### Phase 9: Review with User

Write the generated test file to `e2e/ios-mobile-workflows.spec.ts`. Mark all tasks completed.

**Generate summary from task data:**

```
iOS Workflows to translate: [count]

Workflow: [Name]
  - [N] steps total
  - [N] translatable to WebKit
  - [N] iOS Simulator only ([reason])

Coverage: [N]% of steps can run in CI
Remaining [N]% require ios-workflow-executor for full testing

## Task Summary
- Workflows parsed: [from parse task]
- Selectors found: [from selector task]
- Ambiguous resolved: [count]
- Tests generated: [from generate task]
- File written: [from write task]
```

## Output Files

| File | Purpose |
|------|---------|
| `e2e/ios-mobile-workflows.spec.ts` | Primary: generated WebKit mobile tests |
| `e2e/ios-mobile-workflows.selectors.ts` | Optional: extracted selectors |
| `.claude/ios-workflow-test-mapping.json` | Optional: diff tracking |
