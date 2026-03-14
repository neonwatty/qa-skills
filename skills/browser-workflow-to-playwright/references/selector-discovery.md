# Selector Discovery Prompts

When exploring the codebase for Playwright selectors, use these search patterns for different element types.

## Buttons

```
Search: "button" + "[text from workflow]"
Look for: data-testid, aria-label, className, onClick handler name
```

## Inputs

```
Search: "input" + "[field name]" OR "TextField" + "[label]"
Look for: name, id, placeholder, aria-label, data-testid
```

## Modals/Dialogs

```
Search: "Modal" OR "Dialog" + "[title from workflow]"
Look for: Component name, aria-labelledby, className
```

## Navigation

```
Search: "Link" OR "NavLink" OR "router" + "[destination]"
Look for: href, to prop, data-testid
```

## Selector Priority (Best to Worst)

1. `data-testid="..."` -- Most stable, explicitly for testing
2. `aria-label="..."` -- Accessible and meaningful
3. `role="..." + text` -- Semantic and readable
4. `:has-text("...")` -- Works but fragile if text changes
5. `.class-name` -- Works but fragile if styles change
6. Complex CSS path -- Last resort, very fragile

## Explore Agent Prompt

```
Task tool parameters:
- subagent_type: "Explore"
- model: "sonnet" (balance of speed and thoroughness)
- prompt: |
    You are finding reliable Playwright selectors for browser workflow steps.

    ## Workflows to Find Selectors For
    [Include parsed workflow steps that need selectors]

    ## What to Search For

    For each step, find the BEST available selector using this priority:

    **Selector Priority (best to worst):**
    1. data-testid="..."     <- Most stable, explicitly for testing
    2. aria-label="..."      <- Accessible and meaningful
    3. role="..." + text     <- Semantic and readable
    4. :has-text("...")      <- Works but fragile if text changes
    5. .class-name           <- Works but fragile if styles change
    6. Complex CSS path      <- Last resort, very fragile

    ## Search Strategy

    1. **Component Selectors**
       - Use Grep to search for React/Vue component names mentioned in steps
       - Find data-testid attributes: `data-testid=`
       - Find CSS class names in component files

    2. **Text-Based Selectors**
       - Match button text to actual button implementations
       - Find aria-labels: `aria-label=`
       - Locate placeholder text for inputs

    3. **Structural Selectors**
       - Identify form structures for input fields
       - Find modal/dialog patterns
       - Locate navigation elements

    ## Return Format

    Return a structured mapping:
    ```
    ## Selector Mapping

    ### Workflow: [Name]

    | Step | Element Description | Recommended Selector | Confidence | Notes |
    |------|---------------------|---------------------|------------|-------|
    | 1.1  | Login button        | [data-testid="login-btn"] | High | Found in LoginForm.tsx:45 |
    | 1.2  | Email input         | input[name="email"] | High | Found in LoginForm.tsx:23 |
    | 2.1  | Submit button       | button:has-text("Submit") | Medium | No data-testid, using text |

    ### Ambiguous Selectors (need user input)
    - Step 3.2 "settings button": Found multiple matches:
      1. [data-testid="settings-icon"] in Header.tsx
      2. [data-testid="settings-btn"] in Sidebar.tsx
      - Recommendation: Ask user which one

    ### Missing Selectors (not found)
    - Step 4.1 "export dropdown": Could not find element, may need manual inspection
    ```
```
