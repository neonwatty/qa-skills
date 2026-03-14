# Mobile UX Pattern Test Template

## Test File Structure

```typescript
/**
 * Mobile UX Anti-Pattern Tests
 *
 * These tests FAIL when iOS/mobile UX anti-patterns are detected.
 * Reference: Apple HIG, Material Design vs iOS differences
 */

import { test, expect, Page } from '@playwright/test';

// Viewport sizes
const IPHONE_14 = { width: 393, height: 852 };

// Apple's minimum touch target
const IOS_MIN_TOUCH_TARGET = 44;

// Helper to enter the app (customize per project)
async function enterApp(page: Page) {
  await page.goto('/');
  // Add app-specific setup here
}

test.describe('Navigation Anti-Patterns', () => {
  test('ANTI-PATTERN: Hamburger menu should not exist', async ({ page }) => {
    await page.setViewportSize(IPHONE_14);
    await enterApp(page);

    const hamburger = page.locator('.hamburger-btn, [class*="hamburger"]').first();
    const isVisible = await hamburger.isVisible().catch(() => false);

    expect(isVisible, 'iOS anti-pattern: Hamburger menu detected').toBe(false);
  });

  // ... more navigation tests
});

test.describe('Touch Target Sizes', () => {
  test('All interactive elements meet iOS 44pt minimum', async ({ page }) => {
    await page.setViewportSize(IPHONE_14);
    await enterApp(page);

    const buttons = await page.locator('button:visible').all();
    const violations: string[] = [];

    for (const btn of buttons) {
      const box = await btn.boundingBox();
      if (box && (box.width < IOS_MIN_TOUCH_TARGET || box.height < IOS_MIN_TOUCH_TARGET)) {
        violations.push(`Button ${box.width}x${box.height}px`);
      }
    }

    expect(violations.length, `${violations.length} touch targets too small`).toBe(0);
  });
});

// ... more test categories
```

## Expected Output

After running this skill, the user should have:

1. `e2e/mobile-ux-patterns.spec.ts` - Comprehensive anti-pattern tests
2. Updated CI workflow (if needed)
3. Clear documentation of what's being checked
4. Immediate visibility into current anti-patterns
