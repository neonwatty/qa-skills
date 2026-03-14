# WebKit vs iOS Simulator Comparison

## What Playwright WebKit Provides

- Safari's rendering engine (WebKit)
- Mobile viewport emulation
- Touch event simulation
- User agent spoofing

## What Playwright WebKit Cannot Do (Requires Real iOS Simulator)

- Actual iOS Safari behavior (some quirks differ)
- Real device gestures (pinch-to-zoom physics)
- iOS system UI (permission dialogs, keyboards)
- Safe area inset testing on real notched devices
- Native app wrapper behavior (Capacitor, etc.)

## Translation Strategy

Generate tests that approximate iOS behavior in CI, while marking truly iOS-specific tests for the `ios-workflow-executor` skill.

## CI Test Limitations (WebKit approximation)

These require `ios-workflow-executor` for real iOS Simulator testing:
- System permission dialogs
- Real iOS keyboard behavior
- Pinch/zoom gestures
- Safe area insets on notched devices
- iOS share sheet
- Face ID / Touch ID
- Safari-specific CSS quirks

**CI tests cover:** ~70-80% of typical iOS workflows
**iOS Simulator covers:** 100% (but requires manual/local execution)

## Viewport Sizes to Support

```typescript
const IPHONE_SE = { width: 375, height: 667 };
const IPHONE_14 = { width: 393, height: 852 };
const IPHONE_14_PRO_MAX = { width: 430, height: 932 };
const IPAD_MINI = { width: 768, height: 1024 };
```

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
