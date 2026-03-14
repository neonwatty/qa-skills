# Invocation Examples

```
/qa-skills:playwright-executor                              # run all specs
/qa-skills:playwright-executor multi-user                   # multi-user specs only
/qa-skills:playwright-executor browser --project chromium    # browser specs, chromium only
/qa-skills:playwright-executor ios mobile                    # ios and mobile specs
/qa-skills:playwright-executor --fix                         # run all and auto-fix failures
/qa-skills:playwright-executor multi-user --fix              # run multi-user and auto-fix
/qa-skills:playwright-executor e2e/custom*.spec.ts           # arbitrary glob pattern
```

## Argument Parsing Rules

1. Split `$ARGUMENTS` on whitespace
2. Extract `--fix` flag (boolean, remove from args)
3. Extract `--project <value>` pair (remove both tokens from args)
4. Remaining tokens are spec patterns
5. If a token matches a known shorthand (`multi-user`, `browser`, `ios`, `mobile`), expand it to `e2e/{token}*.spec.ts`
6. If a token looks like a glob or file path, use it directly
7. If no spec pattern remains, default to `e2e/*.spec.ts`
