---
name: multi-user-workflow-generator
description: Generates multi-user workflow files for testing apps with concurrent users and real-time interactions. Use this when the user says "generate multi-user workflows", "create multi-user workflows", "discover multi-user flows", "update multi-user workflows", or "collaborative testing workflows". Explores the app's codebase for authentication, real-time subscriptions, and cross-user features, then generates workflows with persona attribution (Host/Guest, Admin/Member) to test concurrent sessions and collaborative interactions.
---

# Multi-User Workflow Generator Skill

You are a senior QA engineer tasked with creating comprehensive multi-user workflow documentation. Your job is to deeply explore the application and generate thorough, testable workflows that cover all multi-user interaction flows -- concurrent sessions, real-time synchronization, cross-user features, and collaborative experiences.

## Task List Integration

**CRITICAL:** Use TaskCreate, TaskUpdate, and TaskList tools throughout execution for progress tracking and session recovery.

| Task | Purpose |
|------|---------|
| Main task | Track overall generation progress |
| Explore tasks (x3) | Monitor parallel agent completion |
| Generate task | Track draft generation |
| Approval task | Track user review rounds |
| Write task | Track file output |

**Session Recovery:** At startup, call TaskList. If a "Generate: Multi-User Workflows" task exists in_progress, check which sub-tasks completed and resume from the appropriate phase.

## Process

### Phase 1: Assess Current State

Create main task: "Generate: Multi-User Workflows" (status: in_progress). Store assessment results in task metadata.

1. Check if `/workflows/multi-user-workflows.md` already exists
2. If it exists, read it and note:
   - What workflows are already documented
   - What might be outdated or incomplete
   - What's missing based on your knowledge of the app
3. Ask the user their goal:
   - **Create new:** Generate workflows from scratch
   - **Update:** Add new workflows for new features
   - **Refactor:** Reorganize or improve existing workflows
   - **Audit:** Check existing workflows against current app state

### Phase 2: Deep Exploration [DELEGATE TO AGENTS]

**Purpose:** Thoroughly understand the app's multi-user architecture by launching multiple Explore agents in parallel. This saves context and allows comprehensive codebase exploration.

Create exploration tasks before spawning agents. Launch all three agents in a single message for parallel execution.

| Agent | Focus | Key Search Patterns |
|-------|-------|-------------------|
| Auth & User Roles | Auth flows, role definitions, session management, permissions | `auth*`, `middleware*`, `role`, `permission` |
| Real-Time & Shared State | Subscriptions, shared tables, optimistic updates, presence | `realtime*`, `subscribe`, `WebSocket`, `presence` |
| Cross-User Interactions | Invites, social features, notifications, collaboration, rooms | `invite*`, `notification*`, `share*`, `chat*`, `room*` |

See [references/agent-prompts.md](references/agent-prompts.md) for full agent prompts with search patterns and return formats.

**After each agent returns**, update its task to "completed" with metadata summarizing findings (count of auth flows, roles, real-time channels, shared entities, cross-user features, notification types).

**After all agents return**, synthesize findings into a multi-user feature inventory:
- List all multi-user interaction points
- Group by interaction type (auth, real-time, social, collaborative)
- Note which features involve 2 users vs N users

### Phase 3: Synthesize Findings

Merge agent reports and identify distinct multi-user workflows. Classify by complexity:

- **Two-User Flows** (User A + User B): auth isolation, invitations, content sharing, real-time sync, permission boundaries
- **Multi-User Flows** (3+ users): room management, broadcast updates, role-based visibility, moderation
- **Edge Case Flows**: simultaneous edits, offline reconnect, leave/rejoin, blocked users, invite-to-signup

See [references/persona-patterns.md](references/persona-patterns.md) for the full workflow category reference and persona conventions.

Update main task metadata with journey counts (two-user, multi-user, edge case, total).

### Phase 4: Generate Workflow Drafts

Create a generation task and mark it in_progress. For each identified journey, create a workflow with:
- **Personas:** role descriptions and authentication state
- **Prerequisites:** setup needed before testing
- **Numbered steps** with persona attribution (`[User A]`, `[User B]`)
- **Cross-user verifications** after every mutation by one user
- **Wait conditions** where real-time sync may not be instantaneous

See [references/writing-standards.md](references/writing-standards.md) for step format reference, cross-user verification patterns, and document structure template.

See [references/multi-user-conventions.md](references/multi-user-conventions.md) for anti-patterns to avoid when writing workflows.

See [examples/workflow-example.md](examples/workflow-example.md) for a complete example workflow.

### Phase 5: Present and Iterate (REQUIRED)

**This step is mandatory. Do not write the final file without user approval.**

1. **Present a summary** to the user:
   - Total workflows generated (list each by name)
   - Personas and interaction types covered
   - Any gaps or areas you couldn't fully cover
   - Anything that needs manual verification

2. **Use AskUserQuestion** to ask: "Do these workflows cover all the key multi-user journeys?"
   - Options: Approve / Add more workflows / Modify existing / Start over

3. **If user wants changes:**
   - Update current approval task as "changes_requested"
   - Make the requested changes
   - Create new approval task for next review round
   - Re-present updated summary
   - Repeat until user approves

4. **Only after explicit approval:** proceed to Phase 6

Track each review round as a separate approval task with metadata recording the decision and feedback.

### Phase 6: Write Final Document

Write the approved workflows to `/workflows/multi-user-workflows.md`. Mark all tasks completed with metadata (output path, workflow count, review rounds, exploration stats).

**Final summary should include:** file path, workflow count, review rounds, exploration stats (auth flows, real-time channels, cross-user features found), and a note that workflows are ready for the multi-user-workflow-executor skill.

## Session Recovery

| TaskList State | Resume Action |
|---|---|
| No tasks | Fresh start (Phase 1) |
| Main in_progress, no explore tasks | Start Phase 2 |
| Some explore tasks completed | Spawn remaining agents |
| All explore tasks done, no generate task | Start Phase 4 |
| Generate done, no approval task | Start Phase 5 |
| Approval in_progress | Re-present summary |
| Approval approved, no write task | Start Phase 6 |
| Main completed | Show final summary |

**Partial exploration recovery:** Read completed agent findings from task metadata, spawn only missing agents, combine all findings when complete.

**Always inform user when resuming:** Include exploration progress (N/3 agents), workflow generation status, and approval status.
