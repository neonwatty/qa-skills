# Fix Mode & Verification Agent Prompts

## Fix Agent Prompt (Phase 7)

Spawn one agent per issue. For independent issues, spawn agents in parallel.

```
Task tool parameters (for each issue):
- subagent_type: "general-purpose"
- model: "opus" (thorough code analysis and modification)
- prompt: |
    You are fixing a specific issue found during multi-user workflow testing.

    ## Issue to Fix
    **Issue:** [Issue name and description]
    **Severity:** [High/Med/Low]
    **User affected:** [User A / User B / Cross-user]
    **Current behavior:** [What's wrong]
    **Expected behavior:** [What it should do]
    **Sync timing data:** [If applicable]
    **Screenshot references:** [Paths to before screenshots]

    ## Your Task

    1. **Explore the codebase** to understand the implementation
       - Use Glob to find relevant files
       - Use Grep to search for related code
       - Use Read to examine files

    2. **Plan the fix**
       - Identify which files need changes
       - Consider side effects on both user sessions
       - For sync issues, check WebSocket/polling/SSE implementations

    3. **Implement the fix**
       - Make minimal, focused changes
       - Follow existing code patterns
       - Do not refactor unrelated code

    4. **Return a summary:**
    ```
    ## Fix Complete: [Issue Name]

    ### Changes Made
    - [File 1]: [What changed]
    - [File 2]: [What changed]

    ### Files Modified
    - src/components/Room.tsx (MODIFIED)
    - src/lib/realtime.ts (MODIFIED)

    ### Testing Notes
    - [How to verify the fix works]
    - [Any cross-user sync considerations]
    ```

    Do NOT run tests - the main workflow will handle that.
```

## Verification Agent Prompt (Phase 8)

```
Task tool parameters:
- subagent_type: "general-purpose"
- model: "opus" (thorough test analysis and fixing)
- prompt: |
    You are verifying that code changes pass all tests.

    ## Context
    Recent changes were made to fix multi-user workflow issues (real-time sync,
    cross-user assertions, etc.). You need to verify the codebase is healthy.

    ## Your Task

    1. **Run the test suite:**
       ```bash
       # Detect and run appropriate test command
       npm test          # or yarn test, pnpm test
       ```

    2. **If tests fail:**
       - Analyze the failing tests
       - Determine if failures are related to recent changes
       - Fix the broken tests or update them to reflect new behavior
       - Re-run tests until all pass
       - Document what tests were updated and why

    3. **Run linting and type checking:**
       ```bash
       npm run lint      # or eslint, prettier
       npm run typecheck # or tsc --noEmit
       ```

    4. **Run end-to-end tests locally:**
       ```bash
       npm run test:e2e      # common convention
       npx playwright test   # Playwright
       npx cypress run       # Cypress
       ```

    5. **If E2E tests fail:**
       - Analyze the failures (may be related to multi-user changes)
       - Update E2E tests to reflect new behavior
       - Re-run until all pass
       - Document what E2E tests were updated

    6. **Return verification results:**
    ```
    ## Local Verification Results

    ### Test Results
    - Unit tests: PASS/FAIL [count] passed, [count] failed
    - Lint: PASS/FAIL [errors if any]
    - Type check: PASS/FAIL [errors if any]
    - E2E tests: PASS/FAIL [count] passed, [count] failed

    ### Tests Updated
    - [test file 1]: [why updated]
    - [test file 2]: [why updated]

    ### Status: PASS / FAIL
    [If FAIL, explain what's still broken]
    ```
```

## HTML Report Agent Prompt (Phase 9)

```
Task tool parameters:
- subagent_type: "general-purpose"
- model: "haiku" (simple generation task)
- prompt: |
    Generate an HTML report for multi-user workflow fixes.

    ## Data to Include

    **App Name:** [App name]
    **Date:** [Current date]
    **User A:** [identity/role]
    **User B:** [identity/role]
    **Issues Fixed:** [Count]
    **Issues Remaining:** [Count]

    **Fixes Made:**
    [For each fix:]
    - Issue: [Name]
    - User affected: [User A / User B / Cross-user]
    - Before screenshot (User A): workflows/screenshots/{workflow}/before/{file}-userA.png
    - Before screenshot (User B): workflows/screenshots/{workflow}/before/{file}-userB.png
    - After screenshot (User A): workflows/screenshots/{workflow}/after/{file}-userA.png
    - After screenshot (User B): workflows/screenshots/{workflow}/after/{file}-userB.png
    - Files changed: [List]
    - Sync timing improvement: [before ms -> after ms]
    - Why it matters: [Explanation]

    ## Output

    Write the HTML report to: workflows/multi-user-changes-report.html

    Use this template structure:
    - Executive summary with stats and sync performance
    - Before/after screenshot comparisons for each fix (show both User A and User B views)
    - Sync timing improvements table
    - Files changed section
    - "Why this matters" explanations

    Style: Clean, professional, uses system fonts, responsive grid for side-by-side screenshots.

    Return confirmation when complete.
```

## Markdown Report Agent Prompt (Phase 10)

```
Task tool parameters:
- subagent_type: "general-purpose"
- model: "haiku"
- prompt: |
    Generate a Markdown report for multi-user workflow fixes.

    ## Data to Include
    [Same data as HTML report]

    ## Output

    Write the Markdown report to: workflows/multi-user-changes-documentation.md

    Include:
    - Executive summary
    - Before/after comparison table (both user perspectives)
    - Sync timing comparison table
    - Detailed changes for each fix
    - Files changed
    - Technical implementation notes (especially real-time sync changes)
    - Testing verification results

    Return confirmation when complete.
```
