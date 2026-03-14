# Agent Prompts

Full prompts for agents spawned during iOS workflow-to-Playwright translation.

## Phase 3: Selector Discovery Agent

```
Task tool parameters:
- subagent_type: "Explore"
- model: "sonnet" (balance of speed and thoroughness)
- prompt: |
    You are finding reliable Playwright selectors for iOS/mobile workflow steps.
    These selectors will be used in WebKit mobile viewport tests.

    ## Workflows to Find Selectors For
    [Include parsed workflow steps that need selectors]

    ## What to Search For

    For each step, find the BEST available selector using this priority:

    **Selector Priority (best to worst):**
    1. data-testid="..."          -- Most stable
    2. aria-label="..."           -- Accessible
    3. role="..." + text          -- Semantic
    4. .mobile-[component]        -- Mobile-specific classes
    5. :has-text("...")           -- Text-based
    6. Complex CSS path           -- Last resort

    ## Mobile-Specific Search Strategy

    1. **Mobile Navigation Components**
       - Search for bottom nav, tab bars: `bottom-nav`, `tab-bar`, `mobile-nav`
       - Find mobile-specific layouts: `.mobile-only`, `@media` queries
       - Look for touch-optimized components

    2. **Touch Interaction Elements**
       - Find touch-friendly button classes
       - Locate gesture handlers (swipe, drag components)
       - Identify long-press handlers

    3. **iOS-Style Components**
       - Search for iOS picker components
       - Find action sheet / bottom sheet patterns
       - Locate toggle switches vs checkboxes

    4. **Responsive Breakpoints**
       - Identify mobile breakpoint values
       - Find conditionally rendered mobile components

    ## Return Format

    Return a structured mapping:
    ```
    ## Selector Mapping (Mobile)

    ### Workflow: [Name]

    | Step | Element Description | Recommended Selector | Confidence | Mobile Notes |
    |------|---------------------|---------------------|------------|--------------|
    | 1.1  | Bottom nav Guests tab | [data-testid="nav-guests"] | High | Mobile-only component |
    | 1.2  | Guest list item | .guest-item | Medium | Needs .tap() not .click() |
    | 2.1  | Action sheet | [role="dialog"].action-sheet | High | iOS-style sheet |

    ### Mobile-Specific Considerations
    - Component X only renders on mobile viewport
    - Gesture handler found in SwipeableList.tsx - may need approximation

    ### Ambiguous Selectors (need user input)
    - Step 3.2: Found both mobile and desktop versions

    ### Missing Selectors (not found)
    - Step 4.1: Could not find mobile-specific element
    ```
```

## Phase 6: Code Generation Agent

```
Task tool parameters:
- subagent_type: "general-purpose"
- model: "sonnet" (good balance for code generation)
- prompt: |
    You are generating a Playwright E2E test file for iOS/mobile workflows.
    These tests run in WebKit with mobile viewport emulation.

    ## Input Data

    **Workflows:**
    [Include parsed workflow data with names, steps, substeps]

    **Selector Mapping:**
    [Include selector mapping from Phase 3 agent]

    **Existing Test File (if updating):**
    [Include existing test content if this is an update, or "None - new file"]

    ## Your Task

    Generate `e2e/ios-mobile-workflows.spec.ts` with:

    1. **File header** explaining WebKit limitations vs real iOS
    2. **Mobile viewport config** (iPhone 14: 393x852)
    3. **WebKit + touch config** via test.use()
    4. **Helper functions** (swipe, pullToRefresh)
    5. **Test.describe block** for each workflow
    6. **Individual tests** using .tap() for touch interactions
    7. **test.skip** for iOS Simulator-only steps

    ## Mobile-Specific Requirements

    - Use `.tap()` instead of `.click()` for touch interactions
    - Use the swipe helper for swipe gestures
    - Mark pinch/zoom as test.skip (iOS Simulator only)
    - Mark permission dialogs as test.skip
    - Add mobile user agent string
    - Configure hasTouch: true

    ## Handle Special Cases

    - [MANUAL] steps -> `test.skip()` with explanation
    - iOS-only gestures (pinch) -> `test.skip()` with "iOS Simulator only" note
    - Permission dialogs -> `test.skip()` with "requires real iOS"
    - Long press -> `await element.click({ delay: 500 })`

    ## Return Format

    Return the complete test file content ready to write.
    Also return a summary:
    ```
    ## Generation Summary
    - Workflows: [count]
    - Total tests: [count]
    - WebKit translatable: [count]
    - iOS Simulator only: [count]
    - Coverage: [percentage]% can run in CI
    ```
```
