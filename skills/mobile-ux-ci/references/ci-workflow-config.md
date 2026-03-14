# CI Workflow Configuration

## GitHub Actions Workflow

```yaml
name: Mobile UX Checks

on: [push, pull_request]

jobs:
  mobile-ux:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
      - run: npm ci
      - run: npx playwright install chromium
      - run: npm run dev &
      - run: npx playwright test mobile-ux-patterns.spec.ts
```

## Integration Options

1. **Add to existing Playwright CI workflow** - Preferred if Playwright CI already exists
2. **Create new standalone workflow** - Use the template above
3. **Add as a step in existing CI** - Append the test command to an existing job

## Customization Points

When generating tests, ask about:

1. **Severity levels**: Which anti-patterns should fail CI vs warn?
2. **Exceptions**: Any patterns that are intentionally kept?
3. **Additional patterns**: App-specific anti-patterns to detect?
4. **Viewport sizes**: Which devices to test?
