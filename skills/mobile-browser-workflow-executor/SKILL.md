---
name: mobile-browser-workflow-executor
description: Executes mobile browser workflows from /workflows/mobile-browser-workflows.md using Playwright MCP with mobile viewport emulation. Use this when the user says "run mobile browser workflows", "execute mobile browser workflows", "test mobile browser workflows", or "test mobile viewport". Executes workflows in Chrome with iPhone 15 Pro viewport (393x852), captures screenshots with device frames, audits for iOS HIG violations, documents issues, and generates HTML reports.
---

# Mobile Browser Workflow Executor

This skill executes mobile browser workflows defined in `/workflows/mobile-browser-workflows.md` using Playwright MCP as the primary engine, with Claude-in-Chrome as an optional alternative. It emulates a mobile viewport (393x852 - iPhone 14 Pro dimensions), validates workflows step-by-step, audits for iOS Human Interface Guidelines (HIG) anti-patterns, captures visual evidence with device frame mockups, and generates comprehensive reports.

## Execution Modes

- **Audit Mode (Default):** Execute workflows, capture findings, generate reports. Does not modify code.
- **Fix Mode:** Execute audit, then automatically remediate identified issues with user approval.

## Task List Integration

**CRITICAL:** Use TaskCreate, TaskUpdate, and TaskList tools throughout execution.

| Task | Purpose |
|------|---------|
| Master task | `mobile-browser-workflow-execution` - tracks overall session |
| Per-workflow tasks | One task per workflow with step progress |
| Agent tasks | UX evaluation, fix agents, verification, report generation |

### Session Recovery

At skill start, check for existing session file: `.claude/tasks/mobile-browser-workflow-executor-session.json`. If found, prompt user: "Resume previous session or start fresh?"

| Session State | Resume Action |
|---------------|---------------|
| No session exists | Create new, start Phase 1 |
| Workflow execution incomplete | Resume at incomplete workflow/step |
| UX evaluation not started | Start Phase 4 |
| Fix mode incomplete | Resume fix agents |
| Reports not generated | Start Phase 10-11 |

## 12-Phase Execution Flow

### Phase 1: Read Workflows & Initialize

1. Check for existing session state
2. Read `/workflows/mobile-browser-workflows.md`
3. Parse workflow structure (name, URL, steps, expected outcomes)
4. Create master task hierarchy with per-workflow tasks

### Phase 2: Initialize Mobile Browser

Prompt user for engine choice (Playwright MCP recommended, Claude-in-Chrome alternative). Configure viewport (393x852), user agent (iOS Safari 17.0), and touch events.

See [references/playwright-mcp-tools.md](references/playwright-mcp-tools.md) for initialization code and tool reference.
See [references/viewport-config.md](references/viewport-config.md) for device specifications.

### Phase 3: Execute Workflow

Execute workflow steps sequentially with before/after screenshots at each step. Actions map to Playwright tools: Navigate, Tap, Type, Swipe, Wait, Verify, Screenshot.

See [references/playwright-mcp-tools.md](references/playwright-mcp-tools.md) for the action mapping table, step execution template, and touch event simulation code.

For each workflow: navigate to starting URL, execute steps with retry logic (2 retries for timeouts/missing elements), capture screenshots, and scan for obvious iOS HIG issues.

### Phase 4: UX Platform Evaluation [DELEGATE TO AGENT]

Delegate comprehensive iOS HIG validation to the Task tool. The agent evaluates: navigation patterns, touch target sizing (44pt minimum), native component usage, visual design (WCAG AA contrast), and platform conventions.

See [references/agent-prompts.md](references/agent-prompts.md) for the full evaluation agent prompt, anti-pattern checklist, and measurement utilities (JavaScript).

Output: Findings documented in `.claude/plans/mobile-browser-workflow-findings.md`.

### Phase 5: Record Findings

Consolidate all findings from execution and UX evaluation into structured markdown with severity (Critical/High/Medium/Low), category, location, current implementation, iOS HIG violation reference, impact, recommended solution, and implementation code.

### Phase 6: Generate Audit Report

Present summary to user with workflow count, findings by severity, critical issues list, and generated artifacts. Prompt user for next steps:
1. Fix issues automatically (Phase 8)
2. Generate detailed reports (Phase 10-11)
3. Deep dive on specific findings (return to Phase 4)
4. Mark complete (Phase 12)

### Phase 7: Screenshot Management

Organize screenshots into before/after/analysis/mockups directories per workflow. Optimize for web delivery and generate annotated versions highlighting issues.

See [../../references/screenshot-guide.md](../../references/screenshot-guide.md) for directory structure, optimization code, annotation generation, and device frame wrapping.

### Phase 8: Fix Mode Execution [DELEGATE TO AGENTS]

Present fix plan to user with proposed changes grouped by category. After user approval, spawn parallel fix agents (one per independent group: navigation, touch-targets, color-contrast, components). Each agent reads current files, applies iOS HIG-compliant fixes, and validates syntax.

See [references/agent-prompts.md](references/agent-prompts.md) for fix agent prompts and implementation templates.

### Phase 9: Local Verification [DELEGATE TO AGENT]

Re-run workflows to verify fixes resolved issues. Agent captures new screenshots, measures touch targets, verifies navigation changes, checks contrast ratios, and generates before/after metrics comparison.

See [references/agent-prompts.md](references/agent-prompts.md) for the verification agent prompt.

### Phase 10: Generate HTML Report [DELEGATE TO AGENT]

Create self-contained HTML report with embedded CSS, device frame mockups, dark/light mode, interactive expand/collapse findings, and print-friendly styles.

See [examples/audit-report-template.html](examples/audit-report-template.html) for the full HTML template.

### Phase 11: Generate Markdown Report [DELEGATE TO AGENT]

Create GitHub-flavored Markdown report with table of contents, collapsible sections, relative image links, code blocks with syntax highlighting, and emoji status indicators.

### Phase 12: Create PR and Monitor CI

Stage changes, create commit with detailed message listing all fixes by severity, create branch, push, create PR with metrics table and before/after screenshots, and monitor CI checks. Handle visual regression (expected), lint errors (fix immediately), and test failures (investigate).

## Reference Materials

- [references/playwright-mcp-tools.md](references/playwright-mcp-tools.md) - Tool reference for both engines, action mapping, initialization code, step execution templates
- [../../references/automation-limitations.md](../../references/automation-limitations.md) - Known limitations, error handling patterns, best practices
- [../../references/screenshot-guide.md](../../references/screenshot-guide.md) - Directory structure, optimization, annotation, device frame wrapping
- [references/agent-prompts.md](references/agent-prompts.md) - Full prompts for evaluation, fix, verification, and report agents with measurement utilities
- [references/viewport-config.md](references/viewport-config.md) - Device specifications, CSS device frame mockup, session state structure
- [examples/audit-report-template.html](examples/audit-report-template.html) - Complete HTML report template with CSS and JavaScript
