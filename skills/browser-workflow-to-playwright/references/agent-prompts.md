# Agent Prompts Reference

Detailed prompts for agents spawned during Playwright conversion.

## Phase 6: Code Generation Agent

```
Task tool parameters:
- subagent_type: "general-purpose"
- model: "sonnet" (good balance for code generation)
- prompt: |
    You are generating a Playwright E2E test file from browser workflow specifications.

    ## Input Data

    **Workflows:**
    [Include parsed workflow data with names, steps, substeps]

    **Selector Mapping:**
    [Include selector mapping from Phase 3 agent]

    **Existing Test File (if updating):**
    [Include existing test content if this is an update, or "None - new file"]

    ## Your Task

    Generate `e2e/browser-workflows.spec.ts` following this structure:

    1. **File header** with generation timestamp and instructions
    2. **Imports** from @playwright/test
    3. **Test.describe block** for each workflow
    4. **Test.beforeEach** for common setup (navigation)
    5. **Individual tests** for each step
    6. **test.skip** for [MANUAL] steps with clear comments

    ## Code Style Requirements

    - Use the recommended selectors from the mapping
    - Add comments for each substep
    - Include setup code in tests that need prior state
    - Mark ambiguous selectors with TODO comments
    - Follow Playwright best practices

    ## Handle Special Cases

    - [MANUAL] steps -> `test.skip()` with explanation
    - Ambiguous selectors -> Use best guess + TODO comment
    - Missing selectors -> Use descriptive text selector + TODO
    - Steps needing prior state -> Add setup within test

    ## Return Format

    Return the complete test file content ready to write.
    Also return a summary:
    ```
    ## Generation Summary
    - Workflows: [count]
    - Total tests: [count]
    - Skipped (manual): [count]
    - TODOs for review: [count]
    ```
```
