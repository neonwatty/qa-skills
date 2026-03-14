---
name: browser-workflow-generator
description: Generates, creates, or updates browser workflow files. Use this when the user says "generate browser workflows", "create browser workflows", "update browser workflows", "iterate on browser workflows", or "discover browser workflows". Explores the app's codebase to discover all user-facing features, routes, and interactions, then creates comprehensive numbered workflows with substeps covering the full user experience.
---

# Browser Workflow Generator Skill

You are a senior QA engineer tasked with creating comprehensive user workflow documentation. Your job is to deeply explore the application and generate thorough, testable workflows that cover all key user journeys.

## Task List Integration

**CRITICAL:** Use TaskCreate, TaskUpdate, and TaskList tools throughout execution for progress tracking and session recovery.

### Task Hierarchy
```
[Main Task] "Generate: Browser Workflows"
  └── [Explore Task] "Explore: Routes & Navigation" (agent)
  └── [Explore Task] "Explore: Components & Features" (agent)
  └── [Explore Task] "Explore: State & Data" (agent)
  └── [Research Task] "Research: Web UX Conventions" (agent)
  └── [Generate Task] "Generate: Workflow Drafts"
  └── [Approval Task] "Approval: User Review #1"
  └── [Write Task] "Write: browser-workflows.md"
```

### Session Recovery Check
At the start, call TaskList. If a "Generate: Browser Workflows" task exists in_progress, check which subtasks completed and resume from the appropriate phase.

## Process

### Phase 1: Assess Current State

Create main task "Generate: Browser Workflows" and set to in_progress.

1. Check if `/workflows/browser-workflows.md` already exists
2. If it exists, note what workflows are documented and what might be missing
3. Ask the user their goal: **Create new** / **Update** / **Refactor** / **Audit**

Update task metadata with assessment results (existingFile, userGoal).

### Phase 2: Explore the Application [DELEGATE TO AGENTS]

Spawn three Explore agents in parallel to understand the app thoroughly. Create a task for each before spawning.

- **Agent 1 - Routes & Navigation:** Finds all routes, nav patterns, entry points
- **Agent 2 - Components & Features:** Finds all interactive UI components, major features, component patterns
- **Agent 3 - State & Data:** Maps data model, CRUD actions, API patterns

See [references/agent-prompts.md](references/agent-prompts.md) for the full agent prompts and return format specifications.

After each agent returns, mark its task completed with metadata summarizing findings. After all complete, synthesize into a feature inventory grouped by app area.

### Phase 3: Identify User Journeys

Based on exploration, identify key user journeys across three categories:

- **Core Journeys:** Onboarding, primary task completion, main section navigation
- **Feature Journeys:** Each major feature with happy path and key variations
- **Edge Case Journeys:** Error handling, empty states, settings, account management

Update main task metadata with journey counts.

### Phase 4: Research UX Conventions [DELEGATE TO AGENT]

Spawn a general-purpose agent to research web UX conventions for each page type identified in Phase 2/3. The agent searches for reference examples (Dribbble, well-known SaaS apps), visits 2-3 examples per page type, and documents expected conventions and anti-patterns.

See [references/agent-prompts.md](references/agent-prompts.md) for the full UX research agent prompt.

Include UX expectations in workflows so the executor knows what to verify.

### Phase 5: Generate Workflows

For each journey, create a workflow following this structure:

```markdown
## Workflow: [Descriptive Name]

> [Brief description of what this workflow tests and why]

1. [Top-level step]
   - [Substep with specific detail]
   - [Substep with expected outcome]
2. [Next top-level step]
   - [Substep]
3. Verify [expected final state]
```

See [references/writing-standards.md](references/writing-standards.md) for the full step types table and formatting guidelines.

See [references/automation-guidelines.md](references/automation-guidelines.md) for automation-friendly writing tips and `[MANUAL]` tagging.

See [references/web-anti-patterns.md](references/web-anti-patterns.md) for UX verification steps to include.

See [examples/workflow-example.md](examples/workflow-example.md) for a complete example workflow.

### Phase 6: Organize & Write

Structure the final document with sections for Quick Reference table, Core Workflows, Feature Workflows, and Edge Case Workflows. Do not write to file yet -- proceed to Phase 7 for approval.

### Phase 7: Review with User (REQUIRED)

**This step is mandatory. Do not write the final file without user approval.**

Create an approval task. Present a summary of all workflows generated, features covered, gaps, and anything needing manual verification.

Use AskUserQuestion with options: Approve / Add more / Modify / Start over.

If changes requested: update approval task, create new approval task for next round, iterate until approved. Track each review round in task metadata.

Only after explicit approval proceed to Phase 8.

### Phase 8: Write File and Complete

Write to `/workflows/browser-workflows.md`. Mark write task and main task as completed with metadata (outputPath, workflowCount, reviewRounds, explorationAgents, uxResearch).

Present final summary with file path, workflow count, review rounds, and exploration summary from task metadata.

## Handling Updates

When updating existing workflows:
1. **Preserve working workflows** -- don't rewrite what works
2. **Mark deprecated steps** -- note what's outdated if UI changed
3. **Add new workflows** -- append new features as new workflows
4. **Version notes** -- add changelog comments for significant changes

## Session Recovery

If resuming from an interrupted session, use this decision tree:

| TaskList State | Resume Action |
|---|---|
| Main in_progress, no explore tasks | Start Phase 2 |
| Some explore tasks completed | Spawn remaining agents only |
| All explore tasks done, no research | Start Phase 4 |
| Research done, no generate task | Start Phase 5 |
| Generate done, no approval task | Start Phase 7 |
| Approval in_progress | Present summary again |
| Approval approved, no write task | Start Phase 8 |
| Main completed | Show final summary |
| No tasks exist | Fresh start (Phase 1) |

For partial exploration, read completed agent findings from task metadata and spawn only missing agents.

Always inform the user when resuming: show exploration progress, UX research status, workflow count, approval status, and next action.
