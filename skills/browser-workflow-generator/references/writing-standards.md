# Workflow Writing Standards

## Step Types

| Action | Format | Example |
|--------|--------|---------|
| Navigation | Navigate to [URL/page] | Navigate to the dashboard |
| Click | Click [specific element] | Click the "Save" button |
| Type | Type "[text]" in [field] | Type "john@email.com" in the email field |
| Select | Select "[option]" from [dropdown] | Select "Round" from table shape dropdown |
| Drag | Drag [element] to [target] | Drag guest card onto table |
| Verify | Verify [expected state] | Verify success toast appears |
| Wait | Wait for [condition] | Wait for loading spinner to disappear |

## Substep Format

- Use bullet points under numbered steps
- Include specific selectors or descriptions
- Note expected intermediate states

## Guidelines for Writing Steps

- Be specific: "Click the blue 'Add Guest' button in the toolbar" not "Click add"
- Include expected outcomes: "Verify the modal appears" not just "Open modal"
- Use consistent language: Navigate, Click, Type, Verify, Drag, Select
- Group related actions under numbered steps with substeps
- Include wait conditions where timing matters
