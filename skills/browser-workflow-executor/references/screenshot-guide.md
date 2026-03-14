# Screenshot Management Guide

## Directory Structure

```
workflows/
├── screenshots/
│   ├── {workflow-name}/
│   │   ├── before/
│   │   │   ├── 01-hover-states-missing.png
│   │   │   ├── 02-keyboard-nav-broken.png
│   │   │   └── ...
│   │   └── after/
│   │       ├── 01-hover-states-added.png
│   │       ├── 02-keyboard-nav-fixed.png
│   │       └── ...
│   └── {another-workflow}/
│       ├── before/
│       └── after/
├── browser-workflows.md
└── browser-changes-report.html
```

## Naming Convention

- Format: `{NN}-{descriptive-name}.png`
- Examples:
  - `01-hover-states-missing.png` (before)
  - `01-hover-states-added.png` (after)

## Capturing BEFORE Screenshots

1. When an issue is identified during workflow execution
2. Take screenshot BEFORE any fix is applied
3. Save to `workflows/screenshots/{workflow-name}/before/`
4. Use descriptive filename that identifies the issue
5. Record the screenshot path in the issue tracking

## Capturing AFTER Screenshots

1. Only after user approves fixing an issue
2. After fix agent completes, refresh the browser tab
3. Take screenshot showing the fix
4. Save to `workflows/screenshots/{workflow-name}/after/`
5. Use matching filename pattern to the before screenshot

## Audit Report Screenshots

During Phase 3 execution, save screenshots with this convention:
- Path: `workflows/screenshots/browser-audit/wfNN-stepNN.png`
- Naming: `wf{workflow_number:02d}-step{step_number:02d}.png`
- These files will be embedded in the HTML audit report using relative paths
