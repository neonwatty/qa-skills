# Automation-Friendly Workflow Guidelines

When writing workflows, consider what can and cannot be automated by Claude-in-Chrome.

## Prefer UI Actions Over Keyboard Shortcuts

**Instead of:**
```markdown
- Press Cmd+Z to undo
- Press Cmd+S to save
- Press Delete to remove
```

**Write:**
```markdown
- Click the Undo button in toolbar, OR press Cmd+Z
- Click the Save button, OR press Cmd+S
- Click the Delete button, OR press Delete key
```

## Mark Non-Automatable Steps

Use `[MANUAL]` tag for steps that require manual verification:

```markdown
3. Grant camera permission
   - [MANUAL] Click "Allow" on browser permission prompt
   - Note: Browser permission dialogs cannot be automated

4. Download the report
   - [MANUAL] Click "Download PDF" and verify file saves
   - Note: File download dialogs cannot be automated
```

## Known Automation Limitations

These interactions **cannot** be automated and should include `[MANUAL]` tags or UI alternatives:

| Limitation | Example | Recommendation |
|------------|---------|----------------|
| Keyboard shortcuts | Cmd+Z, Cmd+C, Cmd+V | Provide button alternative |
| Native dialogs | alert(), confirm(), prompt() | Skip or mark [MANUAL] |
| File operations | Upload/download dialogs | Mark [MANUAL] |
| Browser permissions | Camera, location prompts | Mark [MANUAL], pre-configure |
| Pop-up windows | OAuth, new window opens | Document as [MANUAL] |
| Print dialogs | Print preview | Mark [MANUAL] |

## Include Prerequisites for Automation

When workflows require specific setup:

```markdown
## Workflow: Upload Profile Photo

**Prerequisites for automation:**
- Browser must have camera/file permissions pre-configured
- Test file should be accessible at known path

> Tests uploading a new profile photo.

1. Open profile settings
   ...
```
