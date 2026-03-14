---
name: browser-workflow-to-playwright
description: Translates browser workflow markdown files into Playwright E2E tests for CI. Use this when the user says "convert workflows to playwright", "translate workflows to CI", "generate playwright tests from workflows", "promote workflows to CI", or "automate browser workflows". Converts human-readable workflow markdown into executable Playwright test files with proper selectors and assertions.
---

# Browser Workflow to Playwright Skill

You are a senior QA automation engineer. Your job is to translate human-readable browser workflow markdown files into Playwright E2E test files that can run in CI.

## Task List Integration

**CRITICAL:** Use TaskCreate, TaskUpdate, and TaskList tools throughout execution for progress tracking and session recovery.

### Task Hierarchy
```
[Main Task] "Convert: Browser Workflows to Playwright"
  └── [Parse Task] "Parse: browser-workflows.md"
  └── [Check Task] "Check: Existing tests"
  └── [Selector Task] "Selectors: Find for all workflows" (agent)
      └── [Ambiguous Task] "Resolve: Settings button selector" (user input needed)
  └── [Generate Task] "Generate: Playwright test file" (agent)
  └── [Approval Task] "Approval: Review generated tests"
  └── [Write Task] "Write: e2e/browser-workflows.spec.ts"
```

### Session Recovery Check
At the start, call TaskList. If a "Convert: Browser Workflows to Playwright" task exists in_progress, check which subtasks completed (parsing, selector discovery, ambiguous resolutions, code generation) and resume from the appropriate phase.

## The Translation Pipeline

```
/workflows/browser-workflows.md     ->     e2e/browser-workflows.spec.ts
     (Human-readable)                         (Playwright tests)
```

Use when the user has refined browser workflows and wants to promote them to CI.

## Process

### Phase 1: Read and Parse Workflows

Create main task "Convert: Browser Workflows to Playwright" and parse task.

1. Read `/workflows/browser-workflows.md`. If missing, inform user and stop.
2. Parse all workflows (each starts with `## Workflow:` or `### Workflow:`)
3. Extract: name, description, URL, numbered steps/substeps, `[MANUAL]` tagged steps

Mark parse task completed with metadata (workflowCount, workflows, totalSteps, manualSteps).

### Phase 2: Check for Existing Tests

Create check task.

1. Look for existing `e2e/browser-workflows.spec.ts`
2. If exists, parse to find which workflows are already translated
3. Determine diff: new workflows to add, modified to update, removed to ask about

Mark check task completed with metadata (existingTestFile, toAdd, toUpdate, toRemove, hasCustomCode).

### Phase 3: Explore Codebase for Selectors [DELEGATE TO AGENT]

Create selector discovery task. Spawn an Explore agent to find reliable selectors for every workflow step.

See [references/selector-discovery.md](references/selector-discovery.md) for the full agent prompt, search patterns, and selector priority.

After agent returns, update selector task with metadata (selectorsFound, highConfidence, ambiguous, missing).

For each ambiguous selector, create a resolution task and ask the user to choose via AskUserQuestion. Mark each resolved with the chosen selector.

For missing selectors, flag for manual verification with TODO comments in the generated code.

### Phase 4: Map Actions to Playwright

Translate natural language workflow steps to Playwright commands using the action mapping table.

See [references/action-mapping.md](references/action-mapping.md) for the full translation table.

### Phase 5: Handle Untranslatable Steps

For steps that cannot be automated (`[MANUAL]` tagged, ambiguous selectors, platform-specific), use appropriate patterns: `test.skip()`, TODO comments, or best-guess selectors.

See [references/untranslatable-patterns.md](references/untranslatable-patterns.md) for code patterns for each case.

### Phase 6: Generate Test File [DELEGATE TO AGENT]

Create code generation task. Spawn a general-purpose agent (sonnet model) to generate the complete Playwright test file from parsed workflows and selector mapping.

See [references/agent-prompts.md](references/agent-prompts.md) for the full code generation agent prompt.

See [examples/translation-example.md](examples/translation-example.md) for a complete workflow-to-Playwright translation example.

Update generation task with metadata (workflowsTranslated, totalTests, skippedManual, todosForReview). Review TODOs with user before writing.

### Phase 7: Handle Updates (Diff Strategy)

When updating existing tests:

1. Parse existing test file for workflow names and `// CUSTOM:` modifications
2. Compare with workflow markdown using content hashing
3. Apply strategy: ADD new, SKIP unchanged, UPDATE modified, ASK about removed
4. Preserve any `// CUSTOM:` code blocks, warn if conflicts exist

### Phase 8: Review with User

Create approval task. Present translation summary showing:
- Workflow counts and per-workflow step/test/manual/TODO breakdown
- Selector resolution stats
- Diff summary if updating existing file

Get explicit approval before writing. After approval, write to `e2e/browser-workflows.spec.ts`. Mark write task and main task completed with metadata (outputPath, workflowsTranslated, totalTests, selectorsResolved).

Present final summary with output path, translation table, selector resolution stats, and next steps (run tests, review TODOs, add to CI).

## Test Independence

Since each step becomes a separate test, ensure independence:

1. Each test sets up its own state (don't rely on previous test)
2. Use `test.beforeEach` for common setup
3. Consider `test.describe.serial` only if order truly matters (discouraged)
4. Add setup steps within tests that need prior state

## Error Handling

| Scenario | Action |
|----------|--------|
| Missing workflow file | Inform user, suggest running generator first |
| Unparseable workflow | Show which failed, ask for clarification |
| No selectors found | List the step, ask user for selector |
| Conflicting selectors | Show options, let user choose |
| Playwright not configured | Offer to set up Playwright config |

## Output Files

- **Primary:** `e2e/browser-workflows.spec.ts` -- the generated test file
- **Optional:** `e2e/browser-workflows.selectors.ts` -- extracted selectors for reuse
- **Optional:** `.claude/workflow-test-mapping.json` -- mapping for diff tracking

## Session Recovery

If resuming from an interrupted session, use this decision tree:

| TaskList State | Resume Action |
|---|---|
| Main in_progress, no parse task | Start Phase 1 |
| Parse done, no check task | Start Phase 2 |
| Check done, no selector task | Start Phase 3 |
| Selectors done, resolve tasks pending | Ask user to resolve remaining selectors |
| All resolved, no generate task | Start Phase 6 |
| Generate done, no approval task | Start Phase 8 |
| Approval done, no write task | Write the file |
| Main completed | Show final summary |
| No tasks exist | Fresh start (Phase 1) |

For partial selector resolution, read completed resolve tasks for user choices, present remaining to user.

Always inform user when resuming: workflows parsed, existing tests found, selectors resolved, code generation status, and next action.
