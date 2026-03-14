# Report Generation Agent Prompts

## Phase 10: HTML Report Agent

```
Task tool parameters:
- subagent_type: "general-purpose"
- model: "haiku" (simple generation task)
- prompt: |
    Generate an HTML report for browser UX compliance fixes.

    ## Data to Include

    **App Name:** [App name]
    **Date:** [Current date]
    **Issues Fixed:** [Count]
    **Issues Remaining:** [Count]

    **Fixes Made:**
    [For each fix:]
    - Issue: [Name]
    - Before screenshot: workflows/screenshots/{workflow}/before/{file}.png
    - After screenshot: workflows/screenshots/{workflow}/after/{file}.png
    - Files changed: [List]
    - Why it matters: [Explanation]

    ## Output

    Write the HTML report to: workflows/browser-changes-report.html

    Use this template structure:
    - Executive summary with stats
    - Before/after screenshot comparisons for each fix
    - Files changed section
    - "Why this matters" explanations

    Style: Clean, professional, uses system fonts, responsive grid for screenshots.

    Return confirmation when complete.
```

## Phase 11: Markdown Report Agent

```
Task tool parameters:
- subagent_type: "general-purpose"
- model: "haiku"
- prompt: |
    Generate a Markdown report for browser UX compliance fixes.

    ## Data to Include
    [Same data as HTML report]

    ## Output

    Write the Markdown report to: workflows/browser-changes-documentation.md

    Include:
    - Executive summary
    - Before/after comparison table
    - Detailed changes for each fix
    - Files changed
    - Technical implementation notes
    - Testing verification results

    Return confirmation when complete.
```
