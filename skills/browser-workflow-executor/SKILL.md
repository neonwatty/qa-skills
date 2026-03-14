---
name: browser-workflow-executor
description: Executes browser-based user workflows from /workflows/browser-workflows.md using Claude-in-Chrome MCP. Use this when the user says "run browser workflows", "execute browser workflows", "test browser workflows", or "audit browser flows". Tests each workflow step by step, captures before/after screenshots, documents issues, and generates HTML reports with visual evidence of fixes.
---

# Browser Workflow Executor Skill

You are a QA engineer executing user workflows in a real browser. Your job is to methodically test each workflow, capture before/after evidence, document issues, and optionally fix them with user approval.

## Task List Integration

**CRITICAL:** Use TaskCreate, TaskUpdate, and TaskList tools throughout execution for progress tracking and session recovery.

### Task Hierarchy
```
[Workflow Task] "Execute: User Login Flow"
  └── [Issue Task] "Issue: Missing hover states on submit button"
  └── [Issue Task] "Issue: Keyboard navigation broken in form"
[Workflow Task] "Execute: Checkout Process"
  └── [Issue Task] "Issue: Back button doesn't preserve cart"
[Fix Task] "Fix: Missing hover states" (created in fix mode)
[Verification Task] "Verify: Run test suite"
[Report Task] "Generate: HTML report"
```

## Execution Modes

### Audit Mode (Default Start)
Execute workflows, identify issues, capture BEFORE screenshots, document without fixing, present findings for review.

### Fix Mode (User-Triggered)
User says "fix this issue" or "fix all". Spawn fix agents, capture AFTER screenshots, verify locally, generate reports, create PR.

**Flow:** Audit Mode -> Find Issues -> Capture BEFORE -> Present to User -> (User triggers fix) -> Fix Mode -> Spawn Agents -> Capture AFTER -> Verify -> Generate Reports -> Create PR

## Process

### Phase 1: Read Workflows and Initialize

**Session recovery:** Call TaskList first. If in_progress workflow tasks exist, inform user and resume from the incomplete workflow.

1. Read `/workflows/browser-workflows.md`. If missing or empty, stop and inform the user.
2. Parse all workflows (each starts with `## Workflow:`)
3. List workflows and ask user which to execute (or all)
4. Create a task for each selected workflow: "Execute: [Workflow Name]"

### Phase 2: Initialize Browser

1. Call `tabs_context_mcp` with `createIfEmpty: true`
2. Store the `tabId` for all subsequent operations
3. Take initial screenshot to confirm browser is ready

See [references/chrome-mcp-tools.md](references/chrome-mcp-tools.md) for the full MCP tool reference.

### Phase 3: Execute Workflow

Set each workflow task to in_progress before starting. For each numbered step:

1. **Announce** the step
2. **Execute** using the appropriate MCP tool (navigate, find+click, type, read_page, drag, scroll, wait)
3. **Screenshot** after each major step, saving to `workflows/screenshots/browser-audit/wfNN-stepNN.png`
4. **Observe** for UI/UX issues, technical problems, console errors
5. **Record** observations before moving to next step

When an issue is found, create an issue task linked to the workflow task with details: workflow, step, description, severity, current vs expected behavior, and screenshot path.

After completing all steps, mark workflow task completed with metadata (issuesFound, stepsPassed, stepsFailed).

### Phase 4: UX Platform Evaluation [DELEGATE TO AGENT]

Spawn a general-purpose agent (opus model) to evaluate the web app against platform UX conventions: navigation patterns, hover/focus states, keyboard navigation, component appropriateness, responsiveness, and accessibility.

See [references/agent-prompts.md](references/agent-prompts.md) for the full UX evaluation agent prompt.

### Phase 5: Record Findings

**CRITICAL:** After completing EACH workflow, immediately append to `.claude/plans/browser-workflow-findings.md`. Include: workflow status, step results, issues found, platform appropriateness, UX notes, technical problems, feature ideas, and screenshot references.

This ensures findings are preserved even if the session is interrupted.

### Phase 6: Generate Audit Report (HTML with Screenshots)

Generate an HTML audit report at `workflows/browser-audit-report.html` with embedded screenshots from Phase 3. Every workflow section MUST include `<img>` tags with relative paths to screenshots.

See [examples/audit-report-template.html](examples/audit-report-template.html) for the required HTML structure.

See [references/screenshot-guide.md](references/screenshot-guide.md) for screenshot directory structure and naming conventions.

Present a text summary showing workflows executed, issues found by severity, report path, and options: "fix all" / "fix 1,3,5" / "done".

### Phase 7: Screenshot Management

Before/after screenshots use matching filenames in separate directories per workflow.

See [references/screenshot-guide.md](references/screenshot-guide.md) for the full directory structure, naming conventions, and capture procedures.

### Phase 8: Fix Mode Execution [DELEGATE TO AGENTS]

When user triggers fix mode:

1. List all issue tasks with before screenshots, ask which to fix
2. Create a fix task for each issue to fix, linked to the original issue task
3. Spawn one agent per issue (in parallel for independent issues), each exploring the codebase and implementing minimal focused changes
4. After all agents complete: refresh browser, capture AFTER screenshots, verify fixes visually

See [references/agent-prompts.md](references/agent-prompts.md) for the full fix agent and verification agent prompts.

### Phase 9: Local Verification [DELEGATE TO AGENT]

Spawn a verification agent to run the test suite, linting, type checking, and E2E tests. If tests fail, the agent analyzes failures, fixes broken tests, and re-runs until all pass.

If PASS: proceed to report generation. If FAIL: review with user, spawn another agent.

### Phase 10-11: Generate Reports [DELEGATE TO AGENTS]

Spawn agents (haiku model) to generate both HTML and Markdown reports with before/after screenshot comparisons, files changed, and explanations.

See [references/report-generation.md](references/report-generation.md) for the full report generation agent prompts.

### Phase 12: Create PR and Monitor CI

Only after local verification passes:

1. Create feature branch `fix/browser-ux-compliance`
2. Stage, commit, and push changes
3. Create PR via `gh pr create` with summary, changes list, testing checklist, and screenshot references
4. Monitor CI -- fix failures, re-run until green
5. Report PR status and final session summary from task metadata

## Guidelines

- **Be methodical:** Execute steps in order, don't skip ahead
- **Be observant:** Note anything unusual, even if the step "passes"
- **Be thorough:** Check console for errors, look for visual glitches
- **Be constructive:** Frame issues as opportunities for improvement
- **Prefer clicks over keys:** Always use UI buttons instead of keyboard shortcuts
- **Delegate to agents:** Use agents for research, fixing, verification, and report generation

See [references/automation-limitations.md](references/automation-limitations.md) for known limitations and workarounds.

## Handling Failures

If a step fails:
1. Take a screenshot of the failure state
2. Check console for errors (`read_console_messages`)
3. Note what went wrong
4. Ask the user: continue, retry, or abort?

Do not silently skip failed steps.

## Session Recovery

**Primary method:** Call TaskList to check workflow, issue, and fix task states.

| TaskList State | Resume Action |
|---|---|
| All workflow tasks completed, no fix tasks | Ask user: "Audit complete. Fix issues?" |
| All workflows done, fix tasks in_progress | Resume fix mode |
| Some workflow tasks pending | Resume from first pending workflow |
| Workflow task in_progress | Read findings file, resume from next step |
| No tasks exist | Fresh start (Phase 1) |

**Fallback:** Read `.claude/plans/browser-workflow-findings.md` to determine which workflows completed, recreate tasks for remaining ones.

Always inform user when resuming: completed workflows, issues found, current state, next action. Do not re-execute passed workflows unless specifically requested.
