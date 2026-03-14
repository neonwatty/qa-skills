# Action Mapping: Workflow Language to Playwright Code

## Core Action Translations

| Workflow Language | Playwright Code |
|-------------------|-----------------|
| "Navigate to [URL]" / "Open Safari and navigate to [URL]" | `await page.goto('URL')` |
| "Tap [element]" | `await page.locator(selector).tap()` |
| "Long press [element]" | `await page.locator(selector).click({ delay: 500 })` |
| "Type '[text]'" | `await page.locator(selector).fill('text')` |
| "Swipe up/down/left/right" | Custom swipe helper (see below) |
| "Pull to refresh" | Custom pull-to-refresh helper |
| "Pinch to zoom" | `test.skip('Pinch gesture requires real mobile device')` |
| "Verify [condition]" | `await expect(...).toBe...(...)` |
| "Wait for [element]" | `await expect(locator).toBeVisible()` |
| "[MANUAL] Grant permission" | `test.skip('Permission dialogs require real mobile device')` |

## Special Case Handling

- `[MANUAL]` steps -> `test.skip()` with explanation
- Mobile-only gestures (pinch) -> `test.skip()` with "real mobile device only" note
- Permission dialogs -> `test.skip()` with "requires real mobile"
- Long press -> `await element.click({ delay: 500 })`

### WebKit (iOS)

- iOS-only gestures (pinch) -> `test.skip()` with "iOS Simulator only" note
- Permission dialogs -> `test.skip()` with "requires real iOS"

### Chromium (Chrome Mobile)

- Mobile-only gestures (pinch) -> `test.skip()` with "real mobile device only" note
- Permission dialogs -> `test.skip()` with "requires real mobile device"

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
  await page.mouse.move(startX + deltas[direction].x, startY + deltas[direction].y, { steps: 10 });
  await page.mouse.up();
}
```

## Pull-to-Refresh Helper

```typescript
async function pullToRefresh(page: Page) {
  await swipe(page, 'down', { startY: 150, distance: 400 });
}
```

## Playwright Config

### WebKit (iOS)

If WebKit mobile project doesn't exist, suggest adding to `playwright.config.ts`:

```typescript
// In playwright.config.ts projects array:
{
  name: 'Mobile Safari',
  use: {
    ...devices['iPhone 14'],
    browserName: 'webkit',
  },
},
```

### Chromium (Chrome Mobile)

If Chromium mobile project doesn't exist, suggest adding to `playwright.config.ts`:

```typescript
// In playwright.config.ts projects array:
{
  name: 'Mobile Chrome',
  use: {
    ...devices['iPhone 15 Pro'],
    browserName: 'chromium',
  },
},
```
