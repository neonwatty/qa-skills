# Untranslatable Step Patterns

## Translatable (Approximate in Mobile Viewport)

- Basic taps and navigation
- Form input
- Scroll/swipe gestures
- Visual verification
- URL navigation
- Mobile layout testing

## Not Translatable (Skip with Note)

Steps that cannot be replicated in Playwright and should be skipped:

### WebKit (iOS)

```typescript
test.skip('Step N: [description]', async () => {
  // iOS SIMULATOR ONLY: This step requires real iOS Simulator
  // Original: "[step text]"
  // Reason: [specific iOS feature needed]
  // Test this via: ios-workflow-executor skill
});
```

### Chromium (Chrome Mobile)

```typescript
test.skip('Step N: [description]', async () => {
  // REAL MOBILE DEVICE ONLY: This step requires real mobile device
  // Original: "[step text]"
  // Reason: [specific mobile feature needed]
  // Test this via: mobile-browser-workflow-executor skill
});
```

## Features Requiring Real Devices

### Shared (Both Platforms)

- System permission dialogs (camera, location, notifications)
- Mobile OS keyboard behavior (autocorrect, suggestions)
- Haptic feedback
- Biometric authentication (Face ID / Touch ID)
- Safe area insets (real device only)
- Native share sheet
- Pinch/zoom gestures (real touchscreen physics)

### WebKit (iOS) Only

- iOS-specific CSS quirks in Safari
- App Store interactions
- iOS share sheet specifics
- Face ID / Touch ID (iOS-specific APIs)

### Chromium (Chrome Mobile) Only

- PWA installation prompts
- Real device orientation changes
- Mobile browser-specific quirks (pull-to-refresh behavior variations)

## Special Case Handling

| Case | Translation |
|------|-------------|
| `[MANUAL]` steps | `test.skip()` with explanation |
| Mobile-only gestures (pinch) | `test.skip()` with "real mobile device only" note |
| Permission dialogs | `test.skip()` with "requires real mobile" |
| Long press | `await element.click({ delay: 500 })` |

## CI Limitations Summary

Always inform user of what CI tests CANNOT cover:

### WebKit (iOS)

```
CI Test Limitations (WebKit approximation):

These require ios-workflow-executor for real iOS Simulator testing:
- System permission dialogs
- Real iOS keyboard behavior
- Pinch/zoom gestures
- Safe area insets on notched devices
- iOS share sheet
- Face ID / Touch ID
- Safari-specific CSS quirks

CI tests cover: ~70-80% of typical iOS workflows
iOS Simulator covers: 100% (but requires manual/local execution)
```

### Chromium (Chrome Mobile)

```
CI Test Limitations (Chromium mobile emulation):

These require mobile-browser-workflow-executor for real mobile device testing:
- System permission dialogs
- Real mobile keyboard behavior
- Pinch/zoom gestures
- Safe area insets on notched devices
- Native share sheet
- Biometric authentication
- Mobile browser-specific quirks
- PWA installation flows

CI tests cover: ~70-80% of typical mobile browser workflows
Real mobile device covers: 100% (but requires manual/local execution)
```

## Update/Diff Strategy

When updating existing test files:

1. Parse existing test file
2. Compare with workflow markdown
3. Add new, update changed, ask about removed
4. Preserve `// CUSTOM:` marked code

## Key Differences Between Platforms

1. **Source file:** `/workflows/ios-workflows.md` vs `/workflows/mobile-browser-workflows.md`
2. **Target file:** `e2e/ios-mobile-workflows.spec.ts` vs `e2e/mobile-browser-workflows.spec.ts`
3. **Browser:** WebKit (iOS) vs Chromium (Chrome Mobile)
4. **Viewport:** iPhone 14 (393x852) vs iPhone 15 Pro (393x852)
5. **Touch config:** hasTouch: true (both platforms)
6. **Gesture handling:** .tap() for touch interactions (both platforms)
