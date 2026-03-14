# Mobile Viewport Configuration for Playwright

## Viewport Sizes

```typescript
const IPHONE_SE = { width: 375, height: 667 };
const IPHONE_15_PRO = { width: 393, height: 852 };
const IPHONE_15_PRO_MAX = { width: 430, height: 932 };
const PIXEL_7 = { width: 412, height: 915 };
const GALAXY_S23 = { width: 360, height: 780 };
```

## Chromium Mobile Configuration

```typescript
// In playwright.config.ts projects array:
{
  name: 'Mobile Chrome',
  use: {
    ...devices['iPhone 15 Pro'],
    // Uses default Chromium (not WebKit)
    browserName: 'chromium',
  },
},
```

**Note:** Unlike iOS workflows which use WebKit, mobile browser workflows use Chromium to match the actual testing environment.

## Touch vs Click

Always use `.tap()` instead of `.click()` for mobile tests:

```typescript
// Preferred for mobile
await page.locator('button').tap();

// Fallback if tap doesn't work
await page.locator('button').click();
```

## Handling Keyboard

Mobile keyboards behave differently:

```typescript
// Fill and close keyboard
await page.locator('input').fill('text');
await page.keyboard.press('Enter'); // Dismiss keyboard

// Or tap outside to dismiss
await page.locator('body').tap({ position: { x: 10, y: 10 } });
```

## Safe Area Handling

Cannot truly test safe areas, but can check CSS:

```typescript
// Check that safe area CSS is present (informational)
const usesSafeArea = await page.evaluate(() => {
  return document.documentElement.style.cssText.includes('safe-area');
});
```

## Responsive Breakpoints

Test that mobile layouts activate correctly:

```typescript
test('Mobile layout activates at correct breakpoint', async ({ page }) => {
  await page.setViewportSize({ width: 393, height: 852 });
  await expect(page.locator('.mobile-nav')).toBeVisible();
  await expect(page.locator('.desktop-nav')).not.toBeVisible();
});
```

## What Playwright Chromium Provides

- Chrome's rendering engine (Chromium/Blink)
- Mobile viewport emulation (iPhone 15 Pro: 393x852)
- Touch event simulation
- User agent spoofing
- Mobile-specific gestures (.tap(), swipe helpers)
- hasTouch: true configuration

## What Playwright Chromium Cannot Do

- Actual mobile device behavior (some touch quirks differ)
- Real device gestures (pinch-to-zoom physics on actual touchscreens)
- Mobile OS system UI (permission dialogs, native keyboards)
- Safe area inset testing on real notched devices
- PWA installation flows
- Mobile-specific browser features (pull-to-refresh might vary)
