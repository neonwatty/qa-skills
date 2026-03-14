# Translation Example

## Source: Mobile Browser Workflow Markdown

```markdown
## Workflow: Mobile Guest Assignment

> Tests assigning guests to tables on mobile Chrome.

**URL:** http://localhost:5173/

1. Open app on mobile
   - Navigate to http://localhost:5173/
   - Wait for app to load
   - Verify mobile layout is active

2. Navigate to guest view
   - Tap bottom nav "Guests" tab
   - Verify guest list appears

3. Assign guest to table
   - Long press on a guest name
   - Drag to table (or tap assign button)
   - Verify guest is assigned

4. [MANUAL] Test pinch-to-zoom on canvas
   - This requires real mobile device gesture testing
```

## Generated: Playwright Test

```typescript
test.describe('Workflow: Mobile Guest Assignment', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('http://localhost:5173/');
    await page.waitForLoadState('networkidle');
  });

  test('Step 1: Open app on mobile', async ({ page }) => {
    // Substep: Wait for app to load
    await expect(page.locator('[data-testid="app-container"]')).toBeVisible();

    // Substep: Verify mobile layout is active
    await expect(page.locator('.mobile-layout, .bottom-nav')).toBeVisible();
  });

  test('Step 2: Navigate to guest view', async ({ page }) => {
    // Substep: Tap bottom nav "Guests" tab
    await page.locator('.bottom-nav-item:has-text("Guests")').tap();

    // Substep: Verify guest list appears
    await expect(page.locator('[data-testid="guest-list"]')).toBeVisible();
  });

  test('Step 3: Assign guest to table', async ({ page }) => {
    // Setup: Navigate to guests first
    await page.locator('.bottom-nav-item:has-text("Guests")').tap();
    await expect(page.locator('[data-testid="guest-list"]')).toBeVisible();

    // Substep: Long press on a guest name
    const guest = page.locator('.guest-item').first();
    await guest.click({ delay: 500 }); // Long press approximation

    // Substep: Tap assign button (drag not fully supported)
    await page.locator('[data-testid="assign-btn"]').tap();

    // Substep: Verify guest is assigned
    await expect(page.locator('.guest-item.assigned')).toBeVisible();
  });

  test.skip('Step 4: [MANUAL] Test pinch-to-zoom on canvas', async () => {
    // REAL MOBILE DEVICE ONLY: Pinch gesture cannot be automated in Playwright
    // Test via: mobile-browser-workflow-executor skill with actual mobile browser
    // Original: "Test pinch-to-zoom on canvas"
  });
});
```

## Translation Notes

- **Step 1:** Standard navigation and verification -- fully translatable
- **Step 2:** Tap interaction using `.tap()` -- fully translatable
- **Step 3:** Long press approximated with `click({ delay: 500 })`, drag replaced with tap on assign button -- partially translatable
- **Step 4:** Pinch gesture -- not translatable, marked as `test.skip`

## Coverage Summary

- 4 steps total
- 3 translatable to Chromium mobile (75%)
- 1 real mobile device only (25%)
