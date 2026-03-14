---
name: ios-workflow-generator
description: Generates, creates, or updates iOS workflow files for testing web apps in Safari on the iOS Simulator. Use this when the user says "generate ios workflows", "create ios workflows", "update ios workflows", "iterate on ios workflows", or "test on ios". Explores the web app's codebase to discover all user-facing features, pages, and interactions, then creates iOS-specific numbered workflows with HIG compliance checks covering the full mobile Safari user experience.
---

# iOS Workflow Generator Skill

You are a senior QA engineer creating comprehensive workflow documentation for testing **web applications in Safari on the iOS Simulator**. Explore the web app codebase deeply and generate thorough, testable workflows that verify correct behavior and iOS UX conventions in mobile Safari.

**Important:** These web apps are intended to become **PWAs or wrapped native apps** (Capacitor, Tauri, etc.) and should feel **indistinguishable from native iOS apps**. The UX bar is native iOS quality, not just "mobile-friendly web."

See [../../references/ios-hig-requirements.md](../../references/ios-hig-requirements.md) for the full native iOS feel requirements.
See [../../references/ios-hig-anti-patterns.md](../../references/ios-hig-anti-patterns.md) for the complete anti-pattern tables.

## Task List Integration

**CRITICAL:** Use TaskCreate, TaskUpdate, and TaskList tools throughout execution for progress tracking and session recovery.

### Task Hierarchy

| Task | Purpose |
|------|---------|
| `Generate: iOS Workflows` | Main task tracking overall progress |
| `Explore: Pages & Navigation` | Agent: codebase exploration |
| `Explore: UI Components & Interactions` | Agent: codebase exploration |
| `Explore: Data & State` | Agent: codebase exploration |
| `Research: iOS HIG Conventions` | Agent: UX research |
| `Generate: Workflow Drafts` | Draft creation |
| `Approval: User Review #N` | User approval (may repeat) |
| `Write: ios-workflows.md` | Final file write |

### Session Recovery

At skill start, call `TaskList` to check for existing tasks. If a main task exists with `in_progress`, check child task states and resume from the appropriate phase.

| Task State | Resume Action |
|-----------|---------------|
| No tasks exist | Fresh start (Phase 1) |
| Main in_progress, no explore tasks | Start Phase 2 |
| Some explore tasks complete | Spawn remaining agents |
| All explore complete, no research | Start Phase 4 |
| Research complete, no generate | Start Phase 5 |
| Generate complete, no approval | Start Phase 7 |
| Approval in_progress | Re-present summary |
| Approval approved, no write | Start Phase 8 |
| Main completed | Show final summary |

Always inform user: "Resuming: Exploration [N]/3 complete, HIG Research [status], Workflows [count/pending], Approval [status]"

## Process

### Phase 1: Assess Current State

Create main task `"Generate: iOS Workflows"` and mark `in_progress`.

1. Check if `/workflows/ios-workflows.md` exists; note existing workflows
2. Ask the user their goal: **Create new** / **Update** / **Refactor** / **Audit**

Store assessment in task metadata: `existingFile`, `existingWorkflowCount`, `userGoal`.

### Phase 2: Explore the Web Application [DELEGATE TO AGENTS]

Create three exploration tasks, then spawn three Explore agents in parallel using the Task tool (all in a single message). Each agent explores a different dimension of the app.

| Agent | Focus | Key Outputs |
|-------|-------|-------------|
| Pages & Navigation | Routes, nav components, entry points | Page table, nav structure, base URL |
| UI Components & Interactions | Interactive elements, touch targets, gestures | Component inventory, touch considerations |
| Data & State | State management, CRUD operations, APIs | Entity table, user actions, persistence |

See [references/agent-prompts.md](references/agent-prompts.md) for the full agent prompt templates.

After each agent returns, mark its task `completed` with summary metadata. After all return, synthesize into a feature inventory grouped by app section.

### Phase 3: Identify User Journeys

Based on exploration, identify three categories of journeys:

- **Core Journeys:** Initial load, primary task completion, main navigation
- **Feature Journeys:** Each major feature with happy path and key variations
- **Edge Case Journeys:** Error handling, empty states, settings, offline behavior, permissions

Store journey counts in main task metadata.

### Phase 4: Research UX Conventions [DELEGATE TO AGENT]

Spawn a general-purpose agent to research iOS HIG conventions for each screen type identified. The agent searches for reference examples (Airbnb, Spotify, Instagram patterns), documents expected conventions, and flags anti-patterns.

See [references/agent-prompts.md](references/agent-prompts.md) for the full HIG research agent prompt.

Include iOS UX expectations in workflows so the executor knows what to verify per screen type.

### Phase 5: Generate Workflows

For each journey, create a workflow with: name, description, URL, numbered steps with substeps, expected outcomes, and iOS convention verification steps.

See [../../references/mobile-writing-standards.md](../../references/mobile-writing-standards.md) for the step types table and formatting conventions.
See [../../references/mobile-automation-guidelines.md](../../references/mobile-automation-guidelines.md) for automation limitations and `[MANUAL]` tagging.
See [examples/workflow-example.md](examples/workflow-example.md) for a complete example workflow.

**Guidelines for writing steps:**
- Be specific: "Tap the blue 'Add' button in the top-right corner" not "Tap add"
- Include expected outcomes: "Verify the modal sheet slides up"
- Use consistent action vocabulary: Open, Tap, Type, Verify, Swipe, Long press, Wait, Scroll
- Note accessibility labels when available
- Group related actions under numbered steps with bullet substeps
- Include wait conditions where animations or loading matters

### Phase 6: Organize & Draft

Structure the document with: header (app name, date, base URL, platform), quick reference table, then Core / Feature / Edge Case workflow sections.

**Do not write to file yet -- proceed to Phase 7 for user approval first.**

### Phase 7: Review with User (REQUIRED)

**Mandatory. Do not write the final file without user approval.**

Present a summary: total workflows, screens/features covered, iOS HIG checks included, gaps or areas needing manual verification.

Use `AskUserQuestion` with options: **Approve** / **Add more workflows** / **Modify existing** / **Start over**.

If changes requested, iterate and create a new approval task for each round. Only after explicit approval proceed to writing.

### Phase 8: Write File and Complete

Write the approved workflows to `/workflows/ios-workflows.md`. Mark all tasks completed.

**Final summary from task data:**

```
## iOS Workflows Generated

**File:** /workflows/ios-workflows.md
**Workflows:** [count]
**Review rounds:** [count]
**Base URL:** [URL]

### Exploration Summary
- Pages found: [count]
- UI components found: [count]
- Data entities: [count]

### iOS HIG Research
- Screen types researched: [count]
- Conventions documented: [count]
- Anti-patterns to check: [count]

### Workflows Created
[List of workflow names]

The workflows are ready to be executed with the ios-workflow-executor skill.
```
