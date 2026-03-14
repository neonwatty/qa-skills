# Fix Mode Agent Prompt

When a failing test is determined to be an app bug (not a test bug), spawn a fix agent:

```
Task tool parameters:
- subagent_type: "general-purpose"
- model: "opus"
- prompt: |
    You are fixing an application bug caught by a Playwright E2E test.

    ## Failing Test
    File: [spec file path]
    Test: [test name]
    Error: [error message]

    ## Test Source
    [Include the relevant test code]

    ## Your Task
    1. Read the application source code that the test exercises
    2. Identify the root cause of the failure
    3. Fix the application code (not the test)
    4. Return a summary of changes made

    Do NOT run tests -- the main workflow handles re-running.
```
