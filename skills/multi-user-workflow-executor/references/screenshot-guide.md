# Screenshot Management Guide

## Directory Structure

```
workflows/
├── screenshots/
│   ├── multi-user-audit/
│   │   ├── wf01-step01-userA.png
│   │   ├── wf01-step01-userB.png
│   │   ├── wf01-step02-userA.png
│   │   └── ...
│   ├── {workflow-name}/
│   │   ├── before/
│   │   │   ├── 01-sync-delay-userA.png
│   │   │   ├── 01-sync-delay-userB.png
│   │   │   ├── 02-auth-failure-userB.png
│   │   │   └── ...
│   │   └── after/
│   │       ├── 01-sync-fixed-userA.png
│   │       ├── 01-sync-fixed-userB.png
│   │       ├── 02-auth-fixed-userB.png
│   │       └── ...
│   └── {another-workflow}/
│       ├── before/
│       └── after/
├── multi-user-workflows.md
└── multi-user-audit-report.html
```

## Naming Conventions

- **Audit screenshots:** `wf{NN}-step{NN}-user{A|B}.png`
- **Before/after screenshots:** `{NN}-{descriptive-name}-user{A|B}.png`
- Examples:
  - `01-sync-delay-userA.png` (before, User A view)
  - `01-sync-delay-userB.png` (before, User B view)
  - `01-sync-fixed-userA.png` (after, User A view)
  - `01-sync-fixed-userB.png` (after, User B view)

## Capturing BEFORE Screenshots

1. When an issue is identified during workflow execution
2. Take screenshot from BOTH browsers BEFORE any fix is applied
3. Save to `workflows/screenshots/{workflow-name}/before/`
4. Use descriptive filename that identifies the issue and which user's perspective
5. Record the screenshot paths in the issue tracking

## Capturing AFTER Screenshots

1. Only after user approves fixing an issue
2. After fix agent completes, refresh BOTH browser tabs
3. Take screenshots from BOTH browsers showing the fix
4. Save to `workflows/screenshots/{workflow-name}/after/`
5. Use matching filename pattern to the before screenshots

## Screenshot Capture Methods

- **User A (Chrome MCP):** `computer({ action: 'screenshot', tabId })`
- **User B (Playwright MCP):** `browser_snapshot` or screenshot tools

Save screenshots to disk during execution:
```
Save to: workflows/screenshots/multi-user-audit/wfNN-stepNN-userX.png
```
