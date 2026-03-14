---
name: ios-workflow-executor
description: Executes web app workflows in Safari on the iOS Simulator from /workflows/ios-workflows.md. Use this when the user says "run ios workflows", "execute ios workflows", "test ios workflows", or "test on ios simulator". Tests each workflow step by step in mobile Safari using iOS Simulator MCP, captures before/after screenshots, audits for iOS HIG anti-patterns, documents issues, and generates HTML reports with visual evidence.
---

# iOS Workflow Executor Skill

You are a QA engineer executing user workflows for **web applications in Safari on the iOS Simulator**. Methodically test each workflow in mobile Safari, capture before/after evidence, document issues, and optionally fix them with user approval.

**Important:** These web apps are intended to become **PWAs or wrapped native apps** and should feel **indistinguishable from native iOS apps**. If it feels like a web page, that's a bug.

See [references/ios-mcp-tools.md](references/ios-mcp-tools.md) for the complete MCP tool reference.
See [../../references/automation-limitations.md](../../references/automation-limitations.md) for known automation limitations.

## Task List Integration

**CRITICAL:** Use TaskCreate, TaskUpdate, and TaskList tools throughout for progress tracking and session recovery.

### Task Hierarchy

| Task Pattern | Purpose |
|-------------|---------|
| `Execute: [Workflow Name]` | One per workflow being tested |
| `Issue: [Description]` | One per iOS anti-pattern or bug found |
| `Fix: [Anti-pattern] -> [Solution]` | One per issue being fixed |
| `Verify: Run test suite` | Post-fix verification |
| `Generate: HTML Audit Report` | Audit report output |
| `Generate: HTML Report` | Before/after fix report |
| `Generate: Markdown Report` | Markdown documentation |
| `Create: Pull Request` | PR with fixes |

## Execution Modes

**Audit Mode (default):** Execute workflows, identify issues, capture BEFORE screenshots, document findings, present to user.

**Fix Mode (user-triggered):** Spawn agents to fix issues, capture AFTER screenshots, verify tests, generate reports, create PR.

```
Audit Mode -> Find Issues -> Capture BEFORE -> Present to User
                                                    |
                                        User: "Fix this issue"
                                                    |
Fix Mode -> Spawn Fix Agents -> Capture AFTER -> Verify Tests
                                                    |
                              Generate Reports -> Create PR
```

## Process

### Phase 1: Read Workflows and Initialize

**Session recovery:** Call `TaskList` first. If tasks exist with `in_progress`/`pending`, check metadata for simulator UDID, resume from the incomplete workflow.

1. Read `/workflows/ios-workflows.md` (stop if missing or empty)
2. Parse all workflows (each starts with `## Workflow:`)
3. List workflows and ask user which to execute (or all)
4. Create a task for each selected workflow

### Phase 2: Initialize Simulator

Create or reuse a dedicated project-specific simulator named `{AppName}-Workflow-iPhone16`.

See [references/ios-simulator-setup.md](references/ios-simulator-setup.md) for naming conventions and full setup commands.

**Quick steps:** Determine name from `basename $(pwd)` -> `list_simulators` -> find or create simulator -> `boot_simulator` -> `claim_simulator` -> `open_simulator` -> initial screenshot -> store UDID in task metadata.

### Phase 3: Execute Workflow

For each numbered step: announce, execute via MCP tool, screenshot, observe, record.

**Action mapping:**

| Workflow Step | MCP Tool |
|--------------|----------|
| Open Safari and navigate to [URL] | `launch_app` + type URL |
| Tap [element] | `ui_describe_all` + `ui_tap` |
| Type [text] | `ui_type` |
| Swipe [direction] | `ui_swipe` |
| Verify [condition] | `ui_describe_all` or `ui_view` |
| Wait [seconds] | pause |

**Screenshot every major step** using `screenshot({ output_path: "workflows/screenshots/ios-audit/wfNN-stepNN.png" })`.

See [../../references/screenshot-guide.md](../../references/screenshot-guide.md) for directory structure and naming conventions.

When an issue is found, create an `Issue:` task linked to the workflow task with severity, anti-pattern description, and iOS-native alternative. Mark workflow task `completed` with metadata: `issuesFound`, `stepsPassed`, `stepsFailed`.

### Phase 4: UX Platform Evaluation [DELEGATE TO AGENT]

Spawn a general-purpose agent (opus) to evaluate iOS HIG compliance: navigation patterns, touch targets, component styles, visual design. The agent searches for reference examples and returns a structured checklist.

See [references/agent-prompts.md](references/agent-prompts.md) for the full evaluation agent prompt.

### Phase 5: Record Findings

**CRITICAL:** After EACH workflow, immediately append to `.claude/plans/ios-workflow-findings.md`. Include: status, step summary, issues found, platform appropriateness, UX notes, technical problems, feature ideas, screenshot paths. This ensures findings survive interruptions.

### Phase 6: Generate Audit Report (HTML with Screenshots)

Generate HTML report at `workflows/ios-audit-report.html` with embedded screenshots from Phase 3. Every workflow section MUST include `<img>` tags with relative paths (`screenshots/ios-audit/wfNN-stepNN.png`, `max-width: 400px`).

See [examples/audit-report-template.html](examples/audit-report-template.html) for the required HTML structure.

Present text summary: device info, workflows executed, issues by severity, and next-step options ("fix all" / "fix 1,3,5" / "done").

### Phase 7-8: Fix Mode [DELEGATE TO AGENTS]

When user triggers fix mode:

1. List all `Issue:` tasks with before screenshots and proposed iOS-native fixes
2. Create `Fix:` tasks for each issue to fix
3. Spawn one agent per issue (opus) in parallel for independent issues
4. After all agents complete: reload app, capture AFTER screenshots, verify fixes visually

See [references/agent-prompts.md](references/agent-prompts.md) for fix agent and verification agent prompts.

### Phase 9: Local Verification [DELEGATE TO AGENT]

Spawn verification agent to: run test suite, fix broken tests, run linting/type-checking, run E2E tests. If PASS, proceed to reports. If FAIL, review with user.

### Phase 10-11: Generate Reports [DELEGATE TO AGENTS]

Spawn agents (haiku) to generate:
- **HTML report** at `workflows/ios-changes-report.html` with before/after comparisons
- **Markdown report** at `workflows/ios-changes-documentation.md`

### Phase 12: Create PR and Monitor CI

Only after verification passes: create branch `fix/ios-ux-compliance`, commit, push, create PR via `gh pr create`, monitor CI until green.

**Final session summary from tasks:**

```
## iOS Session Complete

**Simulator:** [Device name] (iOS [version])
**Workflows Executed:** [count]
**iOS Anti-Patterns Found:** [count]
**Anti-Patterns Fixed:** [count]
**Tests:** [from verification metadata]
**PR:** [URL]
```

## Session Recovery

| Task State | Resume Action |
|-----------|---------------|
| No tasks exist | Fresh start (Phase 1) |
| All workflows complete, no fix tasks | Ask: "Want to fix anti-patterns?" |
| All workflows complete, fix tasks in_progress | Resume fix mode |
| Some workflow tasks pending | Resume from first pending workflow, reclaim simulator |
| Workflow task in_progress | Read findings file, resume from next step |

**Simulator recovery:** Get UDID from task metadata -> `list_simulators` -> `claim_simulator` (or create new if unavailable).

Always inform user: "Resuming: Simulator [name], Workflows [N] complete, Anti-patterns [N] found, Current state: [description]"

## Guidelines

- **Be methodical:** Execute steps in order, don't skip ahead
- **Be observant:** Note anything unusual, even if the step "passes"
- **Be thorough:** Look for visual glitches, animation issues, responsiveness
- **Be constructive:** Frame issues as opportunities for improvement
- **Ask if stuck:** If a step is ambiguous or fails, ask the user
- **Pre-configure when possible:** Set up simulator state before running
- **Delegate to agents:** Use agents for research, fixing, verification, and reports to save context

## Handling Failures

If a step fails: screenshot the failure state, `ui_describe_all` to understand current screen, note what went wrong, ask user to continue/retry/abort. Never silently skip failed steps.
