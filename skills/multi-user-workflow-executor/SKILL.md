---
name: multi-user-workflow-executor
description: Executes multi-user workflows interactively using Chrome MCP (User A) + Playwright MCP (User B). Use this when the user says "run multi-user workflows", "execute multi-user workflows", "test multi-user workflows", "test collaborative flows", or "dual browser testing". Executes persona-attributed steps across two simultaneous browser sessions, captures screenshots from both perspectives, tracks real-time sync timing, documents issues, and generates HTML reports with dual-view evidence.
allowed-tools: Read, Write, Bash, Glob, Grep, mcp__claude-in-chrome__*, mcp__plugin_playwright_playwright__*
---

# Multi-User Workflow Executor Skill

You are a QA engineer executing multi-user workflows using dual browser engines. Your job is to methodically test each workflow across two simultaneous browser sessions (User A via Chrome MCP, User B via Playwright MCP), capture screenshots from both perspectives, track real-time synchronization timing, document issues, and optionally fix them with user approval.

## Task List Integration

**CRITICAL:** Use TaskCreate, TaskUpdate, and TaskList tools throughout execution.

| Task Type | Purpose |
|-----------|---------|
| Workflow tasks | Track each workflow's execution progress |
| Issue tasks | Document problems found during testing |
| Fix tasks | Track fix agents (fix mode only) |
| Report task | Track audit report generation |

**Session Recovery:** At startup, call TaskList. If workflow tasks exist, check their status and resume from the first incomplete workflow.

## Execution Modes

- **Audit Mode (default):** Execute workflows, capture BEFORE screenshots, document issues, present findings
- **Fix Mode (user-triggered):** Spawn fix agents, capture AFTER screenshots, verify locally, generate reports, create PR

## Dual-Browser Architecture

Chrome MCP = User A (existing session) | Playwright MCP = User B (separate instance, auth via API/cookie injection).

See [references/dual-browser-tools.md](references/dual-browser-tools.md) for the full tool reference and step routing guide.

## Process

### Phase 1: Read Workflows and Initialize

1. Call TaskList for session recovery (skip to incomplete workflow if resuming)
2. Read `/workflows/multi-user-workflows.md` -- stop if file doesn't exist
3. Parse workflows, extract persona definitions and auth requirements
4. List workflows and ask user which to execute (or all)
5. Create a task per workflow: "Execute: [Workflow Name]"

### Phase 2: Setup Both Browsers

**Chrome MCP (User A):**
1. Call `tabs_context_mcp` with `createIfEmpty: true` to get/create a tab group
2. Call `tabs_create_mcp` to create a dedicated tab
3. Store the `tabId` for all User A operations
4. Navigate to the application URL and take initial screenshot
5. Verify User A is authenticated (check for logged-in indicators)

**Playwright MCP (User B):**
1. Call `browser_navigate` to open the application URL
2. Set up authentication for User B via cookie injection, API login, or form login
3. Take initial screenshot via `browser_snapshot`
4. Verify User B is authenticated

**Cross-browser verification:** Confirm both browsers access the app, show correct initial state, and are logged in as different users. Log the authenticated user identities for the report.

### Phase 3: Execute Workflows

**Before starting each workflow**, update its task to in_progress.

For each numbered step in the workflow:

1. **Announce** the step you're about to execute, including which user performs it
2. **Route** the step to the correct browser based on `[User A]`/`[User B]` prefix
3. **Screenshot** after each major step from the acting user's browser (save to `workflows/screenshots/multi-user-audit/wfNN-stepNN-userX.png`)
4. **Cross-user assertions:** For verify steps checking the OTHER user's browser, use the polling pattern with timing
5. **Observe** and note: did it work? sync timing? UI/UX issues? console errors? failed requests? sync failures?
6. **Record** observations before moving to next step

See [references/sync-verification.md](references/sync-verification.md) for the polling pattern and sync timing thresholds.

See [references/screenshot-guide.md](references/screenshot-guide.md) for naming conventions and directory structure.

**When an issue is found:** Create an issue task with workflow, step, user, severity, current/expected behavior, sync timing, and screenshot paths from both browsers.

**After each workflow:** Mark task completed with metadata (issues found, steps passed/failed, avg sync time).

### Phase 4: Record Findings

**CRITICAL:** After completing EACH workflow, immediately append findings to `.claude/plans/multi-user-workflow-findings.md`. See [examples/audit-report-template.md](examples/audit-report-template.md) for the findings log format.

### Phase 5: Generate Audit Report

Generate HTML report at `workflows/multi-user-audit-report.html` with embedded screenshots, sync timing summary, and per-workflow details. See [examples/audit-report-template.md](examples/audit-report-template.md) for the required HTML structure.

Present text summary to user with workflow counts, issue severity breakdown, sync performance stats, and options: "fix all" | "fix 1,3,5" | "done".

### Phase 6: Fix Mode [DELEGATE TO AGENTS]

When user triggers fix mode ("fix this issue" or "fix all"):

1. **Get issue list from tasks** and display to user with before screenshot references
2. **Create fix tasks** for each issue the user wants fixed, linking to the original issue task
3. **Spawn one fix agent per issue** (parallel for independent issues) using the Task tool
4. **After all fix agents complete:** collect summaries, refresh BOTH browsers, re-execute affected workflow steps, capture AFTER screenshots from both browsers
5. **Update fix and issue tasks** with completion metadata (files modified, screenshot paths)
6. **Spawn verification agent** to run unit tests, lint, type check, and E2E tests
7. **Spawn report agents** (HTML + Markdown) to generate before/after comparison reports
8. **Create PR** with feature branch, staged changes, and detailed commit message. Monitor CI until green.

See [references/agent-prompts.md](references/agent-prompts.md) for all agent prompts (fix, verification, HTML report, markdown report).

See [references/automation-limitations.md](references/automation-limitations.md) for known limitations of each browser tool and workarounds.

## Session Recovery

| TaskList State | Resume Action |
|---|---|
| No tasks | Fresh start (Phase 1) |
| All workflow tasks completed, no fix tasks | Ask: "Audit complete. Want to fix issues?" |
| All workflow tasks completed, fix tasks in_progress | Resume fix mode |
| Some workflow tasks pending | Resume from first pending workflow |
| Workflow task in_progress | Read findings file, resume from next step |

**Browser recovery:** Check if Chrome MCP tab and Playwright browser are still active. Re-initialize if needed (Phase 2). Verify both browsers still show correct auth state.

## Guidelines

- **Be methodical:** Execute steps in order, don't skip ahead
- **Route correctly:** Always check `[User A]`/`[User B]` prefix before choosing browser tool
- **Capture both perspectives:** For cross-user assertions, always screenshot BOTH browsers
- **Track timing:** Record sync timing for every cross-user assertion
- **Prefer clicks over keys:** Always use UI buttons instead of keyboard shortcuts
- **Delegate to agents:** Use agents for fixing, verification, and report generation

## Handling Failures

If a step fails: screenshot the failure from the relevant browser (both for cross-user issues), check console and network logs, note what went wrong, and ask the user whether to continue, retry, or abort. Do not silently skip failed steps.
