---
name: playwright-executor
description: Runs Playwright E2E test suites and reports results. Use this when the user says "run playwright tests", "execute e2e tests", "run e2e", "test results", or "playwright test suite". Discovers test files, executes them with filtering options, parses results, presents summaries, and optionally auto-fixes failing tests using the --fix flag.
argument-hint: "[spec-pattern] [--fix]"
---

# Playwright Executor Skill

You are a QA engineer running Playwright E2E test suites and reporting results. Your job is to discover test files, execute them, parse results, present clear summaries, and optionally fix failing tests.

## Task List Integration

**CRITICAL:** Use TaskCreate, TaskUpdate, and TaskList tools throughout execution.

| Task | Purpose |
|------|---------|
| Main task | Track overall execution |
| Discover task | Track spec file discovery |
| Execute task | Track test run |
| Report task | Track result presentation |
| Fix tasks | Track each failing test fix (fix mode only) |

**Session Recovery:** At startup, call TaskList. If a "Run: Playwright Tests" task exists in_progress, check sub-task metadata to determine which phase completed and resume from there.

## Arguments

This skill receives arguments via `$ARGUMENTS`. Parse them to determine which tests to run and whether to enable fix mode.

| Argument | Meaning | Example Command |
|----------|---------|-----------------|
| *(none)* | Run all `e2e/*.spec.ts` | `/playwright-executor` |
| `multi-user` | Run `e2e/multi-user*.spec.ts` | `/playwright-executor multi-user` |
| `browser` | Run `e2e/browser*.spec.ts` | `/playwright-executor browser` |
| `ios` | Run `e2e/ios*.spec.ts` | `/playwright-executor ios` |
| `mobile` | Run `e2e/mobile*.spec.ts` | `/playwright-executor mobile` |
| `--fix` | Enable auto-fix mode after failures | `/playwright-executor --fix` |
| `--project chromium` | Limit to one browser project | `/playwright-executor --project chromium` |
| Arbitrary glob | Run matching specs | `/playwright-executor e2e/auth*.spec.ts` |

Arguments can be combined: `/playwright-executor multi-user --fix --project chromium`

See [examples/invocations.md](examples/invocations.md) for full parsing rules.

## Process

### Phase 1: Discover

**Create the main task and discovery subtask:**
```
TaskCreate:
- subject: "Run: Playwright Tests"
- description: "Execute Playwright E2E test suite and report results"
- activeForm: "Running Playwright tests"

TaskCreate:
- subject: "Discover: e2e/*.spec.ts files"
- description: "Find spec files matching pattern and verify Playwright is installed"
- activeForm: "Discovering test files"

TaskUpdate:
- taskId: [discover task ID]
- status: "in_progress"
```

**Steps:**

1. **Verify Playwright is installed:**
   ```bash
   npx playwright --version
   ```
   If this fails, inform the user: "Playwright is not installed. Run `npm install -D @playwright/test` and `npx playwright install` first."

2. **Read `playwright.config.ts`** (or `.js`) to understand:
   - Which browser projects are configured (chromium, firefox, webkit, mobile)
   - The test directory (`testDir` setting)
   - Any custom reporter configuration
   - Base URL and other relevant settings

3. **Glob for spec files** matching the parsed pattern:
   ```
   Use Glob tool: pattern = "e2e/*.spec.ts" (or parsed pattern)
   ```

4. **Report discovery results to user:**
   ```
   Found [N] spec files:
   - e2e/browser-workflows.spec.ts
   - e2e/ios-workflows.spec.ts
   - e2e/multi-user-workflows.spec.ts
   ...

   Browser projects: chromium, firefox, webkit
   Config: playwright.config.ts
   ```

5. **Update discovery task:**
   ```
   TaskUpdate:
   - taskId: [discover task ID]
   - status: "completed"
   - metadata: {"specCount": N, "specFiles": [...], "projects": [...]}
   ```

### Phase 2: Configure

Build the `npx playwright test` command from the parsed arguments.

**Base command:**
```bash
npx playwright test
```

**Add flags based on arguments:**
- Spec pattern: append file path(s) → `npx playwright test e2e/multi-user*.spec.ts`
- `--project`: append → `npx playwright test --project chromium`
- Always add: `--reporter=list` for readable stdout output
- Consider adding: `--reporter=json --output=playwright-results.json` for structured parsing

**Final command examples:**
```bash
# All tests
npx playwright test --reporter=list

# Filtered
npx playwright test e2e/multi-user*.spec.ts --project chromium --reporter=list

# With JSON output for parsing
npx playwright test --reporter=list,json 2>&1
```

### Phase 3: Execute

**Create and start execution task:**
```
TaskCreate:
- subject: "Execute: npx playwright test"
- description: "Run Playwright with configured flags and capture output"
- activeForm: "Executing Playwright tests"

TaskUpdate:
- taskId: [execute task ID]
- status: "in_progress"
```

**Run the command:**
```bash
npx playwright test [flags] 2>&1
```

Use the Bash tool with a generous timeout (up to 600000ms for large suites). Capture both stdout and stderr.

**Important:** Playwright returns exit code 1 when tests fail. This is expected -- do not treat it as a fatal error. Capture the output regardless of exit code.

**Update execution task:**
```
TaskUpdate:
- taskId: [execute task ID]
- status: "completed"
- metadata: {"exitCode": N, "duration": "Xs"}
```

### Phase 4: Parse Results

Parse the Playwright output to extract structured results.

**From stdout (list reporter), look for:**
- Lines with checkmarks or crosses indicating pass/fail
- Summary line: `N passed`, `N failed`, `N skipped`
- Duration line
- Error messages and stack traces for failures

**Extract these data points:**
- Total test count
- Passed count
- Failed count
- Skipped count
- Total duration
- For each failure:
  - Test name (full title path)
  - Error message
  - Expected vs actual values (if assertion error)
  - File and line number
  - Brief stack trace

**If JSON reporter output is available** (`playwright-results.json`), parse it for more precise data.

### Phase 5: Report

**Create report task:**
```
TaskCreate:
- subject: "Report: Test Results"
- description: "Present formatted test results summary"
- activeForm: "Generating test report"

TaskUpdate:
- taskId: [report task ID]
- status: "in_progress"
```

**Display a clear summary to the user:**

```
## Playwright Test Results

| Metric   | Count |
|----------|-------|
| Total    | 24    |
| Passed   | 20    |
| Failed   | 3     |
| Skipped  | 1     |
| Duration | 45s   |

### Failed Tests

1. **[chromium] multi-user > should sync messages between users**
   Error: Expected 3 messages, received 2
   File: e2e/multi-user-workflows.spec.ts:45

2. **[firefox] browser > should persist form state on back navigation**
   Error: Timeout waiting for selector '.form-field'
   File: e2e/browser-workflows.spec.ts:112

3. **[webkit] ios > should handle touch gestures**
   Error: Element not visible
   File: e2e/ios-workflows.spec.ts:78

### HTML Report
Open: playwright-report/index.html

### Next Steps
- "fix" or "--fix" to auto-fix failing tests
- "rerun" to run tests again
- "done" to end session
```

**Update report task:**
```
TaskUpdate:
- taskId: [report task ID]
- status: "completed"
- metadata: {"total": N, "passed": N, "failed": N, "skipped": N, "duration": "Xs"}
```

**If all tests pass:** Congratulate the user and end.

**If tests fail and `--fix` flag is set:** Proceed to Phase 6 automatically.

**If tests fail without `--fix`:** Present results and ask the user what to do.

### Phase 6: Fix Mode

Fix mode activates when `--fix` was passed as an argument or when the user says "fix" after seeing results.

**For each failing test:**

1. **Create a fix task:**
   ```
   TaskCreate:
   - subject: "Fix: [test name]"
   - description: |
       Fix failing test: [full test title]
       Error: [error message]
       File: [spec file path]:[line number]
       Expected: [expected value]
       Actual: [actual value]
   - activeForm: "Fixing [test name]"

   TaskUpdate:
   - taskId: [fix task ID]
   - status: "in_progress"
   ```

2. **Read the failing test source** to understand what it does

3. **Read the corresponding app source** that the test exercises

4. **Determine the failure type:**
   - **Test bug** (stale selector, wrong assertion, timing issue, outdated expectation)
   - **App bug** (actual application defect the test correctly caught)

5. **For test bugs:** Fix the test directly:
   - Update selectors to match current DOM
   - Fix incorrect assertions
   - Add proper waits for async operations
   - Update expected values to match current behavior

6. **For app bugs:** Spawn a fix agent. See [references/agent-prompts.md](references/agent-prompts.md) for the full prompt.

7. **Update fix task:**
   ```
   TaskUpdate:
   - taskId: [fix task ID]
   - status: "completed"
   - metadata: {"fixType": "test_bug" | "app_bug", "filesModified": [...]}
   ```

**After all fixes are applied:**

1. **Re-run only the previously failing tests:**
   ```bash
   npx playwright test e2e/specific-file.spec.ts --grep "test name" --reporter=list
   ```

2. **Report fix results:**
   ```
   ## Fix Results

   | Test | Fix Type | Status |
   |------|----------|--------|
   | multi-user > sync messages | Test bug (stale selector) | Now passing |
   | browser > persist form state | App bug (missing handler) | Now passing |
   | ios > touch gestures | Test bug (timing) | Still failing |

   Fixed: 2/3
   Still failing: 1 (may need manual investigation)
   ```

3. If any tests still fail after fixes, inform the user and suggest manual investigation.

## Invocation Examples

See [examples/invocations.md](examples/invocations.md) for full invocation examples and argument parsing rules.

## Guidelines

- **Be clear about results:** Always show pass/fail counts prominently
- **Show errors concisely:** Include error messages but trim long stack traces
- **Classify failures:** Distinguish test bugs from app bugs before fixing
- **Minimal fixes:** When fixing, change as little as possible
- **Re-verify:** Always re-run after fixes to confirm they work
- **Respect config:** Honor the project's `playwright.config.ts` settings
- **Timeout awareness:** Large test suites may take several minutes -- use appropriate Bash timeouts
- **HTML report:** Always remind users about `playwright-report/index.html` for detailed results
