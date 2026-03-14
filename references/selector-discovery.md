# Selector Discovery Patterns

## Selector Priority (best to worst)

1. `data-testid="..."`          -- Most stable
2. `aria-label="..."`           -- Accessible
3. `role="..." + text`          -- Semantic
4. `.mobile-[component]`        -- Mobile-specific classes
5. `:has-text("...")`           -- Text-based
6. Complex CSS path              -- Last resort

## Mobile-Specific Search Strategy

### 1. Mobile Navigation Components
- Search for bottom nav, tab bars: `bottom-nav`, `tab-bar`, `mobile-nav`
- Find mobile-specific layouts: `.mobile-only`, `@media` queries
- Look for touch-optimized components

### 2. Touch Interaction Elements
- Find touch-friendly button classes
- Locate gesture handlers (swipe, drag components)
- Identify long-press handlers

### 3. Mobile-Style Components

#### WebKit (iOS)
- Search for iOS picker components
- Find action sheet / bottom sheet patterns
- Locate toggle switches vs checkboxes

#### Chromium (Chrome Mobile)
- Search for mobile picker components
- Find action sheet / bottom sheet patterns
- Locate toggle switches vs checkboxes

### 4. Responsive Breakpoints
- Identify mobile breakpoint values
- Find conditionally rendered mobile components

## Explore Agent Prompt

Use the Task tool to spawn an Explore agent for selector discovery:

```
Task tool parameters:
- subagent_type: "Explore"
- model: "sonnet"
- prompt: |
    You are finding reliable Playwright selectors for mobile workflow steps.
    These selectors will be used in mobile viewport tests.

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

    3. **Mobile-Style Components**
       - Search for mobile picker components
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

    ### Mobile-Specific Considerations
    - Component X only renders on mobile viewport
    - Gesture handler found in SwipeableList.tsx

    ### Ambiguous Selectors (need user input)
    - Step 3.2: Found both mobile and desktop versions

    ### Missing Selectors (not found)
    - Step 4.1: Could not find mobile-specific element
    ```
```

## Agent Return Format

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

## Handling Ambiguous Selectors (BLOCKING)

For each ambiguous selector, create a blocking task that requires user resolution:

```
TaskCreate:
- subject: "Ambiguous: Step [N.M] - [element description]"
- description: |
    BLOCKING: This selector needs user input.

    Step: [step description]
    Options found:
    1. [selector option 1] - [context]
    2. [selector option 2] - [context]

    Which selector should be used for mobile?
- activeForm: "Awaiting selector choice"
```

Present all ambiguous selectors to user at once. Wait for resolution of ALL before proceeding to generation.
