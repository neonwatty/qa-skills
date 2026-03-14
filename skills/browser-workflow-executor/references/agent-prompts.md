# Agent Prompts Reference

Detailed prompts for agents spawned during workflow execution.

## Phase 4: UX Platform Evaluation Agent

```
Task tool parameters:
- subagent_type: "general-purpose"
- model: "opus" (thorough research and evaluation)
- prompt: |
    You are evaluating a web app for web platform UX compliance.

    ## Page Being Evaluated
    [Include current page URL and brief description]

    ## Quick Checklist - Evaluate Each Item

    **Navigation:**
    - Browser back button works correctly
    - URLs reflect current state (deep-linkable)
    - No mobile-style bottom tab bar
    - Navigation works without gestures (click-based)

    **Interactions:**
    - All interactive elements have hover states
    - Keyboard navigation works (Tab, Enter, Escape)
    - Focus indicators are visible
    - No gesture-only interactions for critical features

    **Components:**
    - Uses web-appropriate form components
    - No iOS-style picker wheels
    - No Android-style floating action buttons
    - Modals don't unnecessarily go full-screen

    **Responsive/Visual:**
    - Layout works at different viewport widths
    - No mobile-only viewport restrictions
    - Text is readable without zooming

    **Accessibility:**
    - Color is not the only indicator of state
    - Form fields have labels

    ## Reference Comparison

    Search for reference examples using WebSearch:
    - "web app [page type] design Dribbble"
    - "[well-known web app like Linear/Notion/Figma] [page type] screenshot"

    Visit 2-3 reference examples and compare:
    - Navigation placement and behavior
    - Component types and interaction patterns
    - Hover/focus states

    ## Return Format

    Return a structured report:
    ```
    ## UX Platform Evaluation: [Page Name]

    ### Checklist Results
    | Check | Pass/Fail | Notes |
    |-------|-----------|-------|

    ### Reference Comparison
    - Reference apps compared: [list]
    - Key differences found: [list]

    ### Issues Found
    - [Issue 1]: [Description] (Severity: High/Med/Low)

    ### Recommendations
    - [Recommendation 1]
    ```
```

## Phase 8: Fix Agent (one per issue)

```
Task tool parameters (for each issue):
- subagent_type: "general-purpose"
- model: "opus" (thorough code analysis and modification)
- prompt: |
    You are fixing a specific UX issue in a web application.

    ## Issue to Fix
    **Issue:** [Issue name and description]
    **Severity:** [High/Med/Low]
    **Current behavior:** [What's wrong]
    **Expected behavior:** [What it should do]
    **Screenshot reference:** [Path to before screenshot]

    ## Your Task

    1. **Explore the codebase** to understand the implementation
       - Use Glob to find relevant files
       - Use Grep to search for related code
       - Use Read to examine files

    2. **Plan the fix**
       - Identify which files need changes
       - Consider side effects

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
    - src/components/Button.css (MODIFIED)
    - src/styles/global.css (MODIFIED)

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
    Recent changes were made to fix UX issues. You need to verify the codebase is healthy.

    ## Your Task

    1. **Run the test suite:**
       ```bash
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
       npm run lint
       npm run typecheck
       ```

    4. **Run end-to-end tests locally:**
       ```bash
       npm run test:e2e
       npx playwright test
       npx cypress run
       ```

    5. **If E2E tests fail:**
       - Analyze the failures
       - Update E2E tests to reflect new UI behavior
       - Re-run until all pass

    6. **Return verification results:**
    ```
    ## Local Verification Results

    ### Test Results
    - Unit tests: [count] passed, [count] failed
    - Lint: [errors if any]
    - Type check: [errors if any]
    - E2E tests: [count] passed, [count] failed

    ### Tests Updated
    - [test file 1]: [why updated]

    ### Status: PASS / FAIL
    [If FAIL, explain what's still broken]
    ```
```
