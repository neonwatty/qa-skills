# Workflow Writing Standards

## Step Types

| Action | Format | Example |
|--------|--------|---------|
| Navigate | Navigate to [URL] | Navigate to http://localhost:5173/ |
| Tap | Tap [specific element] | Tap the "Save" button |
| Type | Type "[text]" in [field] | Type "john@email.com" in email field |
| Swipe | Swipe [direction] on [element/screen] | Swipe left on the list item |
| Long press | Long press [element] | Long press the photo thumbnail |
| Verify | Verify [expected state] | Verify success message appears |
| Wait | Wait for [condition] | Wait for loading indicator to disappear |
| Scroll | Scroll [direction] to [element/position] | Scroll down to "Settings" section |

## Step Writing Guidelines

- Be specific: "Tap the blue 'Add' button in the top-right corner" not "Tap add"
- Include expected outcomes: "Verify the modal sheet slides up" not just "Open modal"
- Use consistent language: Navigate, Tap, Type, Verify, Swipe, Long press, Wait
- Use mobile-specific terminology: "Tap" not "Click", "Swipe" not "Scroll"
- Note accessibility labels when available: "Tap button with label 'Submit'"
- Group related actions under numbered steps with substeps
- Include wait conditions where animations or loading matters

## Substep Format

- Use bullet points under numbered steps
- Include accessibility labels or specific selectors when known
- Note expected intermediate states
- Use mobile interaction terminology

## Mobile-Specific Considerations

### Touch Targets

```markdown
4. Verify touch target sizes
   - Verify all interactive elements are at least 44x44pt
   - Verify adequate spacing between tappable elements (8pt minimum)
   - Verify thumb reach zones for primary actions
```

### Safe Areas

```markdown
5. Verify safe area handling
   - Verify content respects iPhone notch/Dynamic Island
   - Verify no important content in top 44pt or bottom 34pt
   - Verify navigation bar accounts for safe area insets
```

### Orientation

```markdown
6. Test orientation changes [MANUAL]
   - [MANUAL] Rotate device to landscape
   - Verify layout adapts correctly
   - Verify navigation remains accessible
   - [MANUAL] Rotate back to portrait
```

## Workflow Structure Template

```markdown
## Workflow: [Descriptive Name]

> [Brief description of what this workflow tests and why it matters]

**URL:** [https://localhost:5173/app or production URL]
**Device:** iPhone 15 Pro (393x852)

1. [Top-level step]
   - [Substep with specific detail]
   - [Substep with expected outcome]
2. [Next top-level step]
   - [Substep]
3. Verify [expected final state]
```

## Document Organization

```markdown
# [Platform] Workflows

> Auto-generated workflow documentation for [App Name]
> Last updated: [Date]
> Base URL: [https://localhost:5173/app or production URL]
> Platform: [Platform description]
> Device: [Device name and viewport]

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

## Platform-Specific Notes

### iOS Simulator (Safari)
- Platform line: "Web app tested in Safari on iOS Simulator"
- "Open" step variant: "Open Safari and navigate to [URL]"
- Document title prefix: "iOS Workflows"

### Mobile Browser (Chrome Viewport)
- Platform line: "Web app tested in Chrome with mobile viewport"
- Device line: "iPhone 15 Pro (393x852)"
- "Navigate" step variant: "Navigate to [URL]"
- Document title prefix: "Mobile Browser Workflows"
