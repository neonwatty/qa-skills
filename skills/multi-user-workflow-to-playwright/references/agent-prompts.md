# Agent Prompts

## Selector Discovery Agent (Phase 3)

```
Task tool parameters:
- subagent_type: "Explore"
- model: "sonnet" (balance of speed and thoroughness)
- prompt: |
    You are finding reliable Playwright selectors for multi-user workflow steps.
    Multiple personas interact with the app, so elements may appear in different
    views or components depending on the user's role.

    ## Workflows to Find Selectors For
    [Include parsed workflow steps with persona attribution]

    ## What to Search For

    For each step, find the BEST available selector using this priority:

    **Selector Priority (best to worst):**
    1. data-testid="..."     <- Most stable, explicitly for testing
    2. getByRole("...")      <- Accessible and semantic
    3. getByText("...")      <- Readable but fragile if text changes
    4. getByTestId("...")    <- Alias for data-testid patterns
    5. CSS selector          <- Last resort, very fragile

    ## Search Strategy

    1. **Component Selectors**
       - Use Grep to search for React/Vue component names mentioned in steps
       - Find data-testid attributes: `data-testid=`
       - Search for role-based patterns: `role=`, `aria-label=`

    2. **Persona-Specific Views**
       - Host/Admin views may have different components than Guest/Member views
       - Search for role-based rendering (e.g., `isAdmin`, `isHost`, `role ===`)
       - Identify shared vs. persona-specific components

    3. **Real-Time Elements**
       - Search for elements that update via WebSocket/SSE
       - Find counters, presence indicators, live feeds
       - These need extended timeout selectors

    4. **Text-Based Selectors**
       - Match button text to actual button implementations
       - Find aria-labels: `aria-label=`
       - Locate placeholder text for inputs

    ## Return Format

    Return a structured mapping:
    ```
    ## Selector Mapping

    ### Workflow: [Name]

    | Step | Persona | Element Description | Recommended Selector | Confidence | Notes |
    |------|---------|---------------------|---------------------|------------|-------|
    | 1.1  | Host    | Create button       | [data-testid="create-btn"] | High | Found in HostDashboard.tsx:45 |
    | 2.1  | Guest   | Join link           | getByRole("link", { name: "Join" }) | High | Found in JoinPage.tsx:23 |
    | 3.1  | Host    | Member count        | getByText(/\d+ watching/) | Medium | Dynamic text, regex needed |

    ### Ambiguous Selectors (need user input)
    - Step 3.2 "member count selector": Found multiple matches:
      1. [data-testid="member-count"] in HostView.tsx
      2. [data-testid="viewer-count"] in SharedHeader.tsx
      - Recommendation: Ask user which one

    ### Missing Selectors (not found)
    - Step 4.1 "notification badge": Could not find element, may need manual inspection
    ```
```

## Code Generation Agent (Phase 5)

```
Task tool parameters:
- subagent_type: "general-purpose"
- model: "sonnet" (good balance for code generation)
- prompt: |
    You are generating a Playwright E2E test file for multi-user workflows using
    multiple browser contexts.

    ## Input Data

    **Workflows:**
    [Include parsed workflow data with persona attribution]

    **Selector Mapping:**
    [Include selector mapping from Phase 3 agent]

    **Existing Test File (if updating):**
    [Include existing test content if this is an update, or "None - new file"]

    ## Your Task

    Generate `e2e/multi-user-workflows.spec.ts` following the multi-context pattern below.

    ## Multi-Context Test Pattern

    Each workflow becomes a `test.describe` block with:
    - `beforeEach`: creates browser contexts per persona, sets up auth
    - `afterEach`: closes all contexts
    - Tests that switch between pages (pageA, pageB, etc.)
    - Cross-context assertions with extended timeouts

    ## Code Style Requirements

    - Use the recommended selectors from the mapping
    - Add comments for each substep with persona attribution
    - Include API helper functions for precondition setup
    - Mark ambiguous selectors with TODO comments
    - Use { timeout: 10000 } for cross-user sync assertions
    - Follow Playwright best practices

    ## Handle Special Cases

    - [MANUAL] steps -> `test.skip()` with explanation
    - Ambiguous selectors -> Use best guess + TODO comment
    - Missing selectors -> Use descriptive text selector + TODO
    - External service steps (email, push) -> `test.skip()` with explanation
    - Steps needing prior state -> Add setup within test or use API helpers

    ## Return Format

    Return the complete test file content ready to write.
    Also return a summary:
    ```
    ## Generation Summary
    - Workflows: [count]
    - Total tests: [count]
    - Personas: [list]
    - Skipped (manual/external): [count]
    - TODOs for review: [count]
    ```
```
