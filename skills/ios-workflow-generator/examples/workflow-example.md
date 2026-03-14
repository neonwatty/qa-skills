# Example Workflow

A complete example showing proper workflow structure with all conventions.

## Workflow: Create New Item

> Tests the complete flow of creating a new item from the home page.

**URL:** http://localhost:5173/

1. Open the app in Safari
   - Open Safari and navigate to http://localhost:5173/
   - Wait for home page to load
   - Verify navigation is visible

2. Navigate to creation flow
   - Tap the "+" button in top-right corner
   - Verify "New Item" modal appears
   - Verify form fields are empty

3. Fill in item details
   - Tap the "Title" text field
   - Type "My Test Item"
   - Tap the "Category" dropdown
   - Select "Personal" from the list
   - Verify selection is shown

4. Save the item
   - Tap "Save" button
   - Wait for modal to close
   - Verify item appears in list
   - Verify item shows "My Test Item" title

5. Verify persistence
   - Refresh the page (pull down or reload)
   - Verify "My Test Item" still appears in list
