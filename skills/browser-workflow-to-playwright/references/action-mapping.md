# Action Mapping: Workflow Language to Playwright

Translation table for converting natural language workflow steps to Playwright commands.

| Workflow Language | Playwright Code |
|-------------------|-----------------|
| "Navigate to [URL]" | `await page.goto('URL')` |
| "Click [element]" | `await page.locator(selector).click()` |
| "Tap [element]" | `await page.locator(selector).click()` |
| "Type '[text]' in [field]" | `await page.locator(selector).fill('text')` |
| "Press Enter" | `await page.keyboard.press('Enter')` |
| "Wait for [element]" | `await expect(page.locator(selector)).toBeVisible()` |
| "Verify [condition]" | `await expect(...).toBe...(...)` |
| "Scroll to [element]" | `await page.locator(selector).scrollIntoViewIfNeeded()` |
| "Hover over [element]" | `await page.locator(selector).hover()` |
| "Select '[option]' from [dropdown]" | `await page.locator(selector).selectOption('option')` |
| "Upload [file]" | `await page.locator(selector).setInputFiles('path')` |
| "Wait [N] seconds" | `await page.waitForTimeout(N * 1000)` |
| "Take screenshot" | `await page.screenshot({ path: '...' })` |
