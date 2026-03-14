# Example: Multi-User Workflow to Playwright Translation

## Generated Test Structure

The generated test file follows this multi-context pattern:

```typescript
/**
 * Multi-User Workflow Tests
 *
 * Auto-generated from /workflows/multi-user-workflows.md
 * Generated: [timestamp]
 *
 * To regenerate: Run multi-user-workflow-to-playwright skill
 * To update workflows: Edit /workflows/multi-user-workflows.md and re-run
 */

import { test, expect, BrowserContext, Page } from '@playwright/test'

// ============================================================================
// API HELPERS
// Use these to set up preconditions quickly instead of driving the UI
// ============================================================================

async function createUserViaAPI(email: string, password: string) {
  const response = await fetch('/api/auth/register', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ email, password }),
  })
  return response.json()
}

async function loginViaAPI(page: Page, email: string, password: string) {
  // TODO: Replace with your app's auth mechanism
  const response = await fetch('/api/auth/login', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ email, password }),
  })
  const { token } = await response.json()
  await page.evaluate((t) => localStorage.setItem('token', t), token)
}

// ============================================================================
// WORKFLOW: Party Create and Join
// Generated from: multi-user-workflows.md
// Last updated: [timestamp]
// ============================================================================

test.describe('Multi-User: Party Create and Join', () => {
  let contextA: BrowserContext
  let contextB: BrowserContext
  let pageA: Page
  let pageB: Page

  test.beforeEach(async ({ browser }) => {
    // Create isolated browser contexts for each persona
    contextA = await browser.newContext()
    contextB = await browser.newContext()
    pageA = await contextA.newPage()
    pageB = await contextB.newPage()

    // Auth setup per persona
    // TODO: Replace with actual auth for your app
    await loginViaAPI(pageA, 'host@example.com', 'password')
    await loginViaAPI(pageB, 'guest@example.com', 'password')
  })

  test.afterEach(async () => {
    await contextA.close()
    await contextB.close()
  })

  test('Step 1: Host creates a party', async () => {
    // Substep: Host navigates to create page
    await pageA.goto('/create')

    // Substep: Host fills in party details
    await pageA.locator('[data-testid="party-name"]').fill('Test Party')

    // Substep: Host clicks create
    await pageA.locator('[data-testid="create-btn"]').click()

    // Substep: Verify party page loads
    await expect(pageA.getByText('Test Party')).toBeVisible()
  })

  test('Step 2: Guest joins the party', async () => {
    // Setup: Host creates a party first (independent test)
    await pageA.goto('/create')
    await pageA.locator('[data-testid="party-name"]').fill('Test Party')
    await pageA.locator('[data-testid="create-btn"]').click()

    // Substep: Guest navigates to join page
    await pageB.goto('/join')

    // Substep: Guest searches for the party
    await pageB.locator('[data-testid="search-input"]').fill('Test Party')

    // Substep: Guest clicks join
    await pageB.locator('[data-testid="join-btn"]').click()

    // Substep: Verify guest sees the party
    await expect(pageB.getByText('Test Party')).toBeVisible()
  })

  test('Step 3: Host sees updated member count', async () => {
    // Setup: Create party and have guest join
    await pageA.goto('/create')
    await pageA.locator('[data-testid="party-name"]').fill('Test Party')
    await pageA.locator('[data-testid="create-btn"]').click()
    await pageB.goto('/join')
    await pageB.locator('[data-testid="search-input"]').fill('Test Party')
    await pageB.locator('[data-testid="join-btn"]').click()

    // Substep: Host checks member count (cross-context assertion)
    // Extended timeout for real-time sync propagation
    await expect(pageA.getByText('2 watching')).toBeVisible({ timeout: 10000 })
  })

  test.skip('Step 4: Guest receives email notification', async () => {
    // SKIP: Requires real email delivery service
    // Original: "Guest receives a confirmation email with party details"
  })
})

// ============================================================================
// WORKFLOW: [Next Workflow Name]
// ============================================================================

// test.describe('Multi-User: [Next Name]', () => {
//   // ... follows same pattern
// })
```
