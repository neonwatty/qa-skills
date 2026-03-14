# Translation Example: Workflow Markdown to Playwright

## Source: Workflow Markdown

```markdown
## Workflow: User Login

> Tests the login flow for existing users.

**URL:** http://localhost:5173/login

1. Navigate to login page
   - Open the login page
   - Verify login form is visible

2. Enter credentials
   - Type "test@example.com" in email field
   - Type "password123" in password field

3. Submit login
   - Click the "Sign In" button
   - Wait for redirect to dashboard
   - Verify user name appears in header
```

## Output: Generated Playwright Test

```typescript
test.describe('Workflow: User Login', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('http://localhost:5173/login');
  });

  test('Step 1: Navigate to login page', async ({ page }) => {
    // Substep: Open the login page
    // (handled in beforeEach)

    // Substep: Verify login form is visible
    await expect(page.locator('[data-testid="login-form"]')).toBeVisible();
  });

  test('Step 2: Enter credentials', async ({ page }) => {
    // Substep: Type "test@example.com" in email field
    await page.locator('input[name="email"]').fill('test@example.com');

    // Substep: Type "password123" in password field
    await page.locator('input[name="password"]').fill('password123');
  });

  test('Step 3: Submit login', async ({ page }) => {
    // Setup: Fill credentials first (tests are independent)
    await page.locator('input[name="email"]').fill('test@example.com');
    await page.locator('input[name="password"]').fill('password123');

    // Substep: Click the "Sign In" button
    await page.locator('button[type="submit"]:has-text("Sign In")').click();

    // Substep: Wait for redirect to dashboard
    await expect(page).toHaveURL(/\/dashboard/);

    // Substep: Verify user name appears in header
    await expect(page.locator('[data-testid="user-name"]')).toBeVisible();
  });
});
```

## Generated Test File Structure

The full output file follows this structure:

```typescript
/**
 * Browser Workflow Tests
 *
 * Auto-generated from /workflows/browser-workflows.md
 * Generated: [timestamp]
 *
 * To regenerate: Run browser-workflow-to-playwright skill
 * To update workflows: Edit /workflows/browser-workflows.md and re-run
 */

import { test, expect } from '@playwright/test';

// ============================================================================
// WORKFLOW: [Workflow Name]
// Generated from: browser-workflows.md
// Last updated: [timestamp]
// ============================================================================

test.describe('Workflow: [Name]', () => {
  test.beforeEach(async ({ page }) => {
    // Common setup for this workflow
    await page.goto('[base-url]');
  });

  test('Step 1: [Description]', async ({ page }) => {
    // Substep: [substep description]
    await page.locator('[selector]').click();

    // Substep: [substep description]
    await expect(page.locator('[selector]')).toBeVisible();
  });

  test.skip('Step N: [MANUAL - Description]', async () => {
    // MANUAL: [reason]
  });
});

// ============================================================================
// WORKFLOW: [Next Workflow Name]
// ============================================================================

test.describe('Workflow: [Next Name]', () => {
  // ...
});
```
