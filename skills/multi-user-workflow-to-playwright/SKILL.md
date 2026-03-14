---
name: multi-user-workflow-to-playwright
description: Translates multi-user workflow markdown files into Playwright E2E tests with multiple browser contexts. Use this when the user says "convert multi-user workflows to playwright", "translate multi-user workflows to CI", "generate multi-user playwright tests", "multi-context testing", or "collaborative CI tests". Converts persona-attributed workflows into Playwright tests where each persona gets an independent browser context with isolated auth and state, enabling concurrent user simulation in CI.
---

# Multi-User Workflow to Playwright Skill

You are a senior QA automation engineer. Your job is to translate human-readable multi-user workflow markdown files into Playwright E2E test files that use multiple browser contexts to simulate concurrent users. Each persona becomes its own browser context with independent auth and state.

## Task List Integration

**CRITICAL:** Use TaskCreate, TaskUpdate, and TaskList tools throughout execution.

| Task | Purpose |
|------|---------|
| Main task | Track overall translation progress |
| Parse task | Track workflow parsing |
| Check task | Track existing test file diff |
| Selector task | Track selector discovery agent |
| Resolve tasks | Track ambiguous selector resolution (BLOCKING) |
| Generate task | Track code generation agent |
| Approval task | Track user review |
| Write task | Track file output |

**Session Recovery:** At startup, call TaskList. If a "Translate Multi-User Workflows to Playwright" task exists in_progress, check sub-task completion and resume from the appropriate phase.

## The Translation Pipeline

```
/workflows/multi-user-workflows.md  ->  e2e/multi-user-workflows.spec.ts
       (Human-readable)                    (Playwright multi-context tests)
```

## Process

### Phase 1: Parse Workflows

Create the main task and parse subtask.

1. Read `/workflows/multi-user-workflows.md` -- stop if file doesn't exist
2. Parse all workflows (each starts with `## Workflow:` or `### Workflow:`), extracting:
   - Name and description
   - **Personas** (e.g., "Host", "Guest", "Admin", "Member") and their auth requirements
   - Prerequisites (accounts, shared resources, invitations)
   - Numbered steps with **persona attribution** (which user performs each step)
   - Any `[MANUAL]` tagged steps
   - Cross-user assertions (one user's action verified from another's view)
3. Mark parse task complete with workflow inventory metadata (workflow count, persona list, total steps, cross-user assertions, manual steps)

### Phase 2: Check for Existing Tests

1. Look for existing `e2e/multi-user-workflows.spec.ts`
2. If exists, determine diff: new workflows to ADD, modified to UPDATE, removed to ASK user about
3. Note any custom code (`// CUSTOM:` comments) to preserve

See [references/action-mapping.md](references/action-mapping.md) for the update strategy and custom code preservation rules.

### Phase 3: Explore Codebase for Selectors [DELEGATE TO AGENT]

**Purpose:** For each workflow step, explore the codebase to find reliable selectors. Multi-user workflows may reference elements from different user views (e.g., host dashboard vs. guest join page). Delegate this to an Explore agent to save context.

Create a selector discovery task. After the agent returns, update with findings metadata (selectors found, high/medium confidence counts, ambiguous count, missing count).

See [references/agent-prompts.md](references/agent-prompts.md) for the full selector discovery agent prompt.

See [references/selector-discovery.md](references/selector-discovery.md) for search patterns organized by element type.

See [references/action-mapping.md](references/action-mapping.md) for selector priority (data-testid > getByRole > getByText > CSS).

### Phase 4: Resolve Ambiguities

For each ambiguous selector, create a BLOCKING resolution task. Present options to user and record their choice in task metadata. For missing selectors, flag with TODO comments.

### Phase 5: Generate Spec File [DELEGATE TO AGENT]

**Purpose:** Generate the Playwright test file using the multi-context pattern. Each workflow becomes a `test.describe` block with `beforeEach` creating browser contexts per persona, `afterEach` closing all contexts, and tests that switch between pages with cross-context assertions using extended timeouts.

Spawn a code generation agent to produce the multi-context test file. After the agent returns, update the generation task with metadata (workflows translated, total tests, personas, skipped/TODO counts).

See [references/agent-prompts.md](references/agent-prompts.md) for the full code generation agent prompt.

See [references/multi-context-patterns.md](references/multi-context-patterns.md) for the multi-context pattern details and anti-patterns to avoid.

See [examples/translation-example.md](examples/translation-example.md) for a complete example of the generated test structure.

See [examples/api-helpers.md](examples/api-helpers.md) for API helper function patterns for precondition setup.

### Phase 6: User Approval

Before writing the file:

1. **Show translation summary** (from task metadata):
   - Workflows to translate with step counts and persona names
   - Selector confidence breakdown (high, medium, user-resolved)
   - Tests generated, skipped (manual/external), and TODOs for review
   - Cross-user assertion count

2. **Resolve any remaining ambiguous selectors** from Phase 4

3. **Show diff** if updating existing file (from check task metadata)

4. **Get explicit approval** before writing

### Phase 7: Write

Write the approved test file to `e2e/multi-user-workflows.spec.ts`. Mark all tasks completed.

**Final summary:** output path, workflow-to-test mapping table, selector resolution stats, next steps (run tests, review TODOs, verify API helpers, add to CI).

## Output Files

- `e2e/multi-user-workflows.spec.ts` - The generated multi-context test file
- `e2e/multi-user-workflows.helpers.ts` - Extracted API helper functions (optional)
- `.claude/multi-user-workflow-test-mapping.json` - Workflow-to-test mapping for diff tracking (optional)

## Error Handling

| Error | Action |
|-------|--------|
| Missing workflow file | Inform user, suggest running multi-user-workflow-generator first |
| Unparseable workflow | Show which workflow failed, ask for clarification |
| No selectors found | List step and persona, ask user for selector |
| Conflicting selectors | Show options, let user choose |
| Persona auth unclear | Ask user how each persona authenticates |
| Playwright not configured | Offer to set up Playwright config |

## Session Recovery

| TaskList State | Resume Action |
|---|---|
| No tasks | Fresh start (Phase 1) |
| Parse completed, no check task | Start Phase 2 |
| Check completed, no selector task | Start Phase 3 |
| Selector completed, resolve tasks pending | Phase 4: ask user to resolve |
| All resolve tasks completed, no generate task | Start Phase 5 |
| Generate completed, no approval task | Start Phase 6 |
| Approval completed, no write task | Phase 7: write the file |
| Main completed | Show final summary |

**Partial selector recovery:** Read completed resolve tasks for user's choices, present remaining ambiguous selectors.

**Always inform user when resuming:** Include workflows parsed, personas found, selectors found, ambiguous resolution progress, and code generation status.
