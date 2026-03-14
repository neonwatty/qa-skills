# Agent Prompts

## Phase 3: Selector Discovery Agent

See [selector-discovery.md](selector-discovery.md) for the full Explore agent prompt.

## Phase 6: Code Generation Agent

```
Task tool parameters:
- subagent_type: "general-purpose"
- model: "sonnet"
- prompt: |
    You are generating a Playwright E2E test file for mobile browser workflows.
    These tests run in Chromium with mobile viewport emulation.

    ## Input Data

    **Workflows:**
    [Include parsed workflow data with names, steps, substeps]

    **Selector Mapping:**
    [Include selector mapping from Phase 3 agent]

    **Existing Test File (if updating):**
    [Include existing test content if this is an update, or "None - new file"]

    ## Your Task

    Generate `e2e/mobile-browser-workflows.spec.ts` with:

    1. **File header** explaining Chromium limitations vs real mobile
    2. **Mobile viewport config** (iPhone 15 Pro: 393x852)
    3. **Chromium + touch config** via test.use()
    4. **Helper functions** (swipe, pullToRefresh)
    5. **Test.describe block** for each workflow
    6. **Individual tests** using .tap() for touch interactions
    7. **test.skip** for real mobile device-only steps

    ## Mobile-Specific Requirements

    - Use `.tap()` instead of `.click()` for touch interactions
    - Use the swipe helper for swipe gestures
    - Mark pinch/zoom as test.skip (real mobile device only)
    - Mark permission dialogs as test.skip
    - Add mobile user agent string (iPhone 15 Pro Safari)
    - Configure hasTouch: true
    - Configure browserName: undefined (uses default Chromium)

    ## Handle Special Cases

    - [MANUAL] steps -> `test.skip()` with explanation
    - Mobile-only gestures (pinch) -> `test.skip()` with "real mobile device only" note
    - Permission dialogs -> `test.skip()` with "requires real mobile"
    - Long press -> `await element.click({ delay: 500 })`

    ## Return Format

    Return the complete test file content ready to write.
    Also return a summary:
    ```
    ## Generation Summary
    - Workflows: [count]
    - Total tests: [count]
    - Chromium translatable: [count]
    - Real mobile only: [count]
    - Coverage: [percentage]% can run in CI
    ```
```

## Complete Test File Template

```typescript
/**
 * Mobile Browser Workflow Tests
 *
 * Auto-generated from /workflows/mobile-browser-workflows.md
 * Generated: [timestamp]
 *
 * These tests run in Playwright Chromium with iPhone 15 Pro mobile viewport.
 * They emulate mobile Chrome behavior with touch support.
 *
 * For full mobile device testing, use the mobile-browser-workflow-executor skill.
 *
 * To regenerate: Run mobile-browser-workflow-to-playwright skill
 */

import { test, expect, Page } from '@playwright/test';

// iPhone 15 Pro viewport
const MOBILE_VIEWPORT = { width: 393, height: 852 };

// Configure for Chromium mobile
test.use({
  viewport: MOBILE_VIEWPORT,
  hasTouch: true,
  userAgent: 'Mozilla/5.0 (iPhone; CPU iPhone OS 17_0 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/17.0 Mobile/15E148 Safari/604.1',
});

// ============================================================================
// HELPERS
// ============================================================================

async function swipe(
  page: Page,
  direction: 'up' | 'down' | 'left' | 'right',
  options?: { startX?: number; startY?: number; distance?: number }
) {
  const viewport = page.viewportSize()!;
  const startX = options?.startX ?? viewport.width / 2;
  const startY = options?.startY ?? viewport.height / 2;
  const distance = options?.distance ?? 300;

  const deltas = {
    up: { x: 0, y: -distance },
    down: { x: 0, y: distance },
    left: { x: -distance, y: 0 },
    right: { x: distance, y: 0 },
  };

  await page.mouse.move(startX, startY);
  await page.mouse.down();
  await page.mouse.move(
    startX + deltas[direction].x,
    startY + deltas[direction].y,
    { steps: 10 }
  );
  await page.mouse.up();
}

async function pullToRefresh(page: Page) {
  await swipe(page, 'down', { startY: 150, distance: 400 });
}

// ============================================================================
// WORKFLOWS
// ============================================================================

test.describe('Workflow: Example Workflow', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('http://localhost:5173/');
    await page.waitForLoadState('networkidle');
  });

  test('Step 1: Example step', async ({ page }) => {
    // Substep: Example action
    await page.locator('[data-testid="example"]').tap();

    // Substep: Example verification
    await expect(page.locator('[data-testid="result"]')).toBeVisible();
  });

  test.skip('Step 2: Mobile device only', async () => {
    // REAL MOBILE DEVICE ONLY: Requires actual mobile browser
    // Test via: mobile-browser-workflow-executor
  });
});
```
