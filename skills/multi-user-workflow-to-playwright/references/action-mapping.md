# Action Mapping & Selector Priority

## Selector Priority

When choosing selectors for multi-user workflow elements, follow this priority order:

| Priority | Selector Type | Example | When to Use |
|----------|--------------|---------|-------------|
| 1 (Best) | `data-testid` | `[data-testid="create-btn"]` | Most stable, explicitly for testing |
| 2 | `getByRole` | `getByRole('button', { name: 'Create' })` | Semantic and accessible |
| 3 | `getByText` | `getByText('2 watching')` | Readable but fragile if text changes |
| 4 | `getByTestId` | `getByTestId('member-count')` | Alias for data-testid patterns |
| 5 (Worst) | CSS selector | `.member-count-badge` | Last resort, very fragile |

For dynamic text (counters, timestamps), prefer regex patterns:
```typescript
await expect(pageA.getByText(/\d+ watching/)).toBeVisible({ timeout: 10000 })
```

## Handling Updates to Existing Tests

When updating existing tests:

1. **Parse existing test file** to extract:
   - Workflow names and their test blocks
   - Persona-to-context mappings
   - Any custom modifications (marked with `// CUSTOM:`)

2. **Compare with workflow markdown:**
   - Hash each workflow's content to detect changes
   - Track workflow names for additions/removals
   - Detect persona changes (added/removed personas)

3. **Update strategy:**

   | Workflow in MD | Workflow in Tests | Action |
   |----------------|-------------------|--------|
   | Present | Missing | ADD new test block |
   | Present | Present (same) | SKIP (no change) |
   | Present | Present (diff) | UPDATE test block |
   | Missing | Present | ASK user: remove or keep? |

4. **Preserve custom code:**
   - Look for `// CUSTOM:` comments
   - Keep custom assertions, API helpers, or setup
   - Warn user if custom code conflicts with updates
   - Preserve custom API helper functions at the top of the file
