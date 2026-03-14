# Multi-User Workflow Writing Standards

## Step Types

| Action | Format | Example |
|--------|--------|---------|
| Navigation | [User X] Navigate to [URL/page] | [User A] Navigate to the dashboard |
| Click | [User X] Click [specific element] | [User B] Click the "Join Room" button |
| Type | [User X] Type "[text]" in [field] | [User A] Type "Hello team" in the message input |
| Select | [User X] Select "[option]" from [dropdown] | [User B] Select "Viewer" from role dropdown |
| Verify | [User X] Verify [expected state] | [User A] Verify User B's avatar appears in the room |
| Verify Sync | [User X] Verify: sees [state] from [User Y]'s action | [User B] Verify: sees new item added by User A |
| Wait | [User X] Wait for [condition] | [User A] Wait for real-time sync indicator |

## Guidelines for Writing Multi-User Steps

- **Always prefix steps with the acting user:** `[User A]`, `[User B]`, etc.
- **Be specific:** "Click the blue 'Add Guest' button in the toolbar" not "Click add"
- **Include cross-user verifications:** After one user mutates, verify the other user sees the update
- **Include expected outcomes:** "Verify the notification badge shows '1'" not just "Check notifications"
- **Use consistent language:** Navigate, Click, Type, Verify, Drag, Select
- **Include wait conditions:** Real-time sync may not be instantaneous -- note where to wait
- **Mark sync checkpoints:** Clearly indicate when steps verify cross-user synchronization

## Cross-User Verification Pattern

After any mutation by one user, include a Verify step for the other user(s):

```markdown
5. [User A] Click "Add Item" -> item appears in list
6. [User A] Verify: new item visible in their list
7. [User B] Verify: sees new item appear in real time (cross-user sync)
```

## Workflow Structure Template

```markdown
## Workflow N: [Descriptive Name]

### Personas
- User A: [Role description] (authenticated/anonymous)
- User B: [Role description] (authenticated/anonymous)

### Prerequisites
- [Setup needed, e.g., "Both users have accounts", "App running at localhost:3000"]

### Steps
1. [User A] Navigate to /path
2. [User A] Click "Button Text" -> expected outcome
3. [User A] Verify: visible assertion
4. [User B] Navigate to /other-path
5. [User B] Enter "value" in field, click "Submit"
6. [User B] Verify: sees expected result
7. [User A] Verify: sees real-time update from User B's action (cross-user sync)
```

## Final Document Structure

```markdown
# Multi-User Workflows

> Auto-generated multi-user workflow documentation for [App Name]
> Last updated: [Date]

## Quick Reference

| Workflow | Personas | Purpose | Steps |
|----------|----------|---------|-------|
| [Name] | [User A + User B] | [Brief] | [Count] |

---

## Two-User Workflows

### Workflow 1: [Name]
...

## Multi-User Workflows

### Workflow N: [Name]
...

## Edge Case Workflows

### Workflow N: [Name]
...
```

## Handling Updates

When updating existing multi-user workflows:

1. **Preserve working workflows** - Don't rewrite what works
2. **Mark deprecated steps** - If API or UI changed, note what's outdated
3. **Add new workflows** - Append new multi-user features as new workflows
4. **Version notes** - Add changelog comments for significant changes
