# Agent Prompts

Full prompts for agents spawned during iOS workflow execution.

## Phase 4: UX Platform Evaluation Agent

```
Task tool parameters:
- subagent_type: "general-purpose"
- model: "opus" (thorough research and evaluation)
- prompt: |
    You are evaluating a web app for iOS HIG (Human Interface Guidelines) compliance.
    The app should feel indistinguishable from a native iOS app.

    ## Screen Being Evaluated
    [Include current screen description and context]

    ## Quick Checklist - Evaluate Each Item

    **Navigation (must feel native):**
    - Uses tab bar for primary navigation (not hamburger menu)
    - Back navigation feels native (swipe gesture or back button)
    - No breadcrumb navigation
    - Modals slide up from bottom like native iOS sheets

    **Touch & Interaction:**
    - All tap targets are at least 44x44pt
    - No hover-dependent interactions
    - Animations feel native (spring physics, smooth)
    - Forms work well with the on-screen keyboard

    **Components (should match native iOS):**
    - Uses iOS-style pickers, not web dropdowns
    - Toggle switches, not checkboxes
    - No Material Design components (FAB, snackbars, etc.)
    - Action sheets and alerts follow iOS patterns

    **Visual Design:**
    - Typography follows iOS conventions (SF Pro feel)
    - Subtle shadows and rounded corners (not Material elevation)
    - Safe area insets respected on notched devices
    - Doesn't look like a "website" - feels like an app

    ## Reference Comparison

    Search for reference examples using WebSearch:
    - "iOS [screen type] design Dribbble"
    - "[well-known iOS app like Airbnb/Spotify/Instagram] [screen type] screenshot"
    - "iOS Human Interface Guidelines [component]"

    Visit 2-3 reference examples and compare:
    - Navigation placement and style (tab bar position, back button)
    - Component types (iOS pickers vs web dropdowns)
    - Layout and spacing (iOS generous whitespace)
    - Animation and transition patterns

    ## Return Format

    Return a structured report:
    ```
    ## iOS HIG Evaluation: [Screen Name]

    ### Checklist Results
    | Check | Pass/Fail | Notes |
    |-------|-----------|-------|

    ### Reference Comparison
    - Reference apps compared: [list]
    - Key differences found: [list]

    ### Issues Found (iOS Anti-Patterns)
    - [Issue 1]: [Description] (Severity: High/Med/Low)
      - Anti-pattern: [What's wrong]
      - iOS-native alternative: [What it should be]

    ### Recommendations
    - [Recommendation 1]
    ```
```

## Phase 8: Fix Mode Agent (per issue)

```
Task tool parameters (for each issue):
- subagent_type: "general-purpose"
- model: "opus" (thorough code analysis and modification)
- prompt: |
    You are fixing a specific iOS UX issue in a web application.
    The app should feel indistinguishable from a native iOS app.

    ## Issue to Fix
    **Issue:** [Issue name and description]
    **Severity:** [High/Med/Low]
    **iOS Anti-Pattern:** [What's wrong - e.g., "hamburger menu"]
    **iOS-Native Solution:** [What it should be - e.g., "bottom tab bar"]
    **Screenshot reference:** [Path to before screenshot]

    ## Your Task

    1. **Explore the codebase** to understand the implementation
       - Use Glob to find relevant files
       - Use Grep to search for related code
       - Use Read to examine files

    2. **Plan the fix**
       - Identify which files need changes
       - May need to create new iOS-style components
       - Consider side effects

    3. **Implement the fix**
       - Make minimal, focused changes
       - Follow existing code patterns
       - Create iOS-native components if needed
       - Do not refactor unrelated code

    4. **Return a summary:**
    ```
    ## Fix Complete: [Issue Name]

    ### iOS Anti-Pattern Replaced
    - Before: [What was wrong]
    - After: [iOS-native solution]

    ### Changes Made
    - [File 1]: [What changed]
    - [File 2]: [What changed]

    ### Files Modified
    - src/components/IOSTabBar.tsx (NEW)
    - src/components/Navigation.tsx (MODIFIED)

    ### Testing Notes
    - [How to verify the fix works]
    ```

    Do NOT run tests - the main workflow will handle that.
```

## Phase 9: Verification Agent

```
Task tool parameters:
- subagent_type: "general-purpose"
- model: "opus" (thorough test analysis and fixing)
- prompt: |
    You are verifying that code changes pass all tests.

    ## Context
    Recent changes were made to fix iOS UX issues. You need to verify the codebase is healthy.

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
       - Analyze the failures (may be related to UI changes)
       - Update E2E tests to reflect new UI behavior
       - Re-run until all pass
       - Document what E2E tests were updated

    6. **Return verification results:**
    ```
    ## Local Verification Results

    ### Test Results
    - Unit tests: [pass/fail] [count] passed, [count] failed
    - Lint: [pass/fail] [errors if any]
    - Type check: [pass/fail] [errors if any]
    - E2E tests: [pass/fail] [count] passed, [count] failed

    ### Tests Updated
    - [test file 1]: [why updated]
    - [test file 2]: [why updated]

    ### Status: PASS / FAIL
    [If FAIL, explain what's still broken]
    ```
```

## Phase 10: HTML Report Generation Agent

```
Task tool parameters:
- subagent_type: "general-purpose"
- model: "haiku" (simple generation task)
- prompt: |
    Generate an HTML report for iOS HIG compliance fixes.

    ## Data to Include

    **App Name:** [App name]
    **Date:** [Current date]
    **Device:** [Simulator device name and iOS version]
    **Issues Fixed:** [Count]
    **Issues Remaining:** [Count]

    **Fixes Made:**
    [For each fix:]
    - Issue: [Name]
    - iOS Anti-Pattern: [What was wrong]
    - iOS-Native Fix: [What it is now]
    - Before screenshot: workflows/screenshots/{workflow}/before/{file}.png
    - After screenshot: workflows/screenshots/{workflow}/after/{file}.png
    - Files changed: [List]
    - Why it matters: [Explanation of iOS HIG compliance]

    ## Output

    Write the HTML report to: workflows/ios-changes-report.html

    Use this template structure:
    - Executive summary with stats
    - Before/after screenshot comparisons for each fix
    - iOS anti-pattern to iOS-native fix explanation
    - Files changed section
    - "Why this matters for iOS users" explanations

    Style: Clean, professional, Apple-style design (SF Pro fonts feel, iOS blue accents).

    Return confirmation when complete.
```

## Phase 11: Markdown Report Generation Agent

```
Task tool parameters:
- subagent_type: "general-purpose"
- model: "haiku"
- prompt: |
    Generate a Markdown report for iOS HIG compliance fixes.

    ## Data to Include
    [Same data as HTML report]

    ## Output

    Write the Markdown report to: workflows/ios-changes-documentation.md

    Include:
    - Executive summary
    - Before/after comparison table with iOS anti-pattern and fix columns
    - Detailed changes for each fix
    - Files changed
    - Technical implementation notes
    - Testing verification results

    Return confirmation when complete.
```
