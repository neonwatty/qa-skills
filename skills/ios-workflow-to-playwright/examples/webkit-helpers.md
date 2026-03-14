# WebKit Helper Code

Helper functions and configuration for Playwright WebKit mobile tests.

## Test File Header Template

```typescript
/**
 * iOS Mobile Workflow Tests
 *
 * Auto-generated from /workflows/ios-workflows.md
 * Generated: [timestamp]
 *
 * These tests run in Playwright WebKit with iPhone viewport.
 * They approximate iOS Safari behavior but cannot fully replicate it.
 *
 * For full iOS testing, use the ios-workflow-executor skill
 * with the actual iOS Simulator.
 *
 * To regenerate: Run ios-workflow-to-playwright skill
 */

import { test, expect, Page } from '@playwright/test';
```

## Mobile Viewport Configuration

```typescript
// iPhone 14 viewport
const MOBILE_VIEWPORT = { width: 393, height: 852 };

// Configure for WebKit mobile
test.use({
  viewport: MOBILE_VIEWPORT,
  // Use WebKit for closest Safari approximation
  browserName: 'webkit',
  // Enable touch events
  hasTouch: true,
  // Mobile user agent
  userAgent: 'Mozilla/5.0 (iPhone; CPU iPhone OS 17_0 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/17.0 Mobile/15E148 Safari/604.1',
});
```

## Swipe Gesture Helper

```typescript
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
```

## Pull-to-Refresh Helper

```typescript
async function pullToRefresh(page: Page) {
  await swipe(page, 'down', { startY: 150, distance: 400 });
}
```

## Output Files

Primary output:
- `e2e/ios-mobile-workflows.spec.ts` - Generated WebKit mobile tests

Optional outputs:
- `e2e/ios-mobile-workflows.selectors.ts` - Extracted selectors
- `.claude/ios-workflow-test-mapping.json` - Diff tracking
