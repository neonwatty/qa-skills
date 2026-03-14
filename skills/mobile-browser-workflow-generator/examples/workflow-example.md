# Example Workflow

## Workflow: Create New Item

> Tests the complete flow of creating a new item from the home page.

**URL:** http://localhost:5173/
**Device:** iPhone 15 Pro (393x852)

1. Navigate to the app
   - Navigate to http://localhost:5173/
   - Wait for home page to load
   - Verify navigation is visible

2. Open creation flow
   - Tap the "+" button in top-right corner
   - Wait for modal animation to complete
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
   - Refresh the page
   - Wait for content to reload
   - Verify "My Test Item" still appears in list

6. Verify mobile UX conventions
   - Verify "Save" button was at least 44x44pt
   - Verify form scrolled when keyboard appeared
   - Verify modal slid up from bottom (iOS pattern)
   - Verify navigation follows iOS conventions
