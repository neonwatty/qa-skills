# Example Workflow: Create New Event with Tables and Guests

This example demonstrates the complete workflow format with descriptive steps, substeps, and verifications.

## Workflow: Create New Event with Tables and Guests

> Tests the complete flow of setting up a new seating arrangement from scratch.

1. Enter the application
   - Navigate to https://app.example.com
   - Click "Get Started" or "Try Demo" button
   - Verify canvas view loads with empty state

2. Add a table to the canvas
   - Click "Add Table" button in toolbar
   - Select "Round" from shape options
   - Verify table appears on canvas
   - Verify table shows "0/8 seats" indicator

3. Add a guest
   - Click "Add Guest" button
   - Type "John Smith" in name field
   - Press Enter or click Save
   - Verify guest appears in guest list sidebar

4. Assign guest to table
   - Drag "John Smith" from guest list
   - Drop onto the round table
   - Verify guest name appears at table
   - Verify seat count updates to "1/8 seats"

5. Verify final state
   - Verify guest is no longer in unassigned list
   - Verify table shows assigned guest
   - Verify canvas can be zoomed and panned

## Document Structure Example

The final `browser-workflows.md` file should follow this structure:

```markdown
# Browser Workflows

> Auto-generated workflow documentation for [App Name]
> Last updated: [Date]

## Quick Reference

| Workflow | Purpose | Steps |
|----------|---------|-------|
| [Name] | [Brief] | [Count] |

---

## Core Workflows

### Workflow: [Name]
...

## Feature Workflows

### Workflow: [Name]
...

## Edge Case Workflows

### Workflow: [Name]
...
```
