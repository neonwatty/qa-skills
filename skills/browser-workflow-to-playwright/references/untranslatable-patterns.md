# Untranslatable Step Patterns

Patterns for handling workflow steps that cannot be directly translated to Playwright.

## [MANUAL] Tagged Steps

```typescript
test.skip('Step N: [description]', async () => {
  // MANUAL: This step requires human intervention
  // Original: "[step text]"
  // Reason: [why it can't be automated]
});
```

## Ambiguous Selectors

```typescript
// TODO: Selector needs verification - found multiple matches
// Options found:
//   1. [data-testid="btn-submit"]
//   2. button:has-text("Submit")
// Using best guess, please verify:
await page.locator('[data-testid="btn-submit"]').click();
```

## Platform-Specific Steps

```typescript
test.skip('Step N: [description]', async () => {
  // SKIP: This step is browser-specific and tested via browser-workflow-executor
  // Original: "[step text]"
});
```
