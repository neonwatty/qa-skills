# Known Automation Limitations

The Claude-in-Chrome browser automation has the following limitations that cannot be automated.

## Cannot Automate (Must Skip or Flag for Manual Testing)

### 1. Keyboard Shortcuts
- System-level shortcuts (Cmd+Z, Cmd+C, Cmd+V, etc.) may cause extension disconnection
- Browser shortcuts that trigger native behavior can interrupt the session
- **Workaround:** Use UI buttons instead of keyboard shortcuts when available

### 2. Native Browser Dialogs
- `alert()`, `confirm()`, `prompt()` dialogs block all browser events
- File upload dialogs (OS-level file picker)
- Print dialogs
- **Workaround:** Skip steps requiring these, or flag for manual testing

### 3. Pop-ups and New Windows
- Pop-ups that open in new windows outside the MCP tab group
- OAuth flows that redirect to external authentication pages
- **Workaround:** Document as requiring manual verification

### 4. System-Level Interactions
- Browser permission prompts (camera, microphone, notifications, location)
- Download dialogs and download management
- Browser settings and preferences pages
- **Workaround:** Pre-configure permissions or skip these steps

## Handling Limited Steps

When a workflow step involves a known limitation:

1. **Mark as [MANUAL]:** Note the step requires manual verification
2. **Try UI Alternative:** If testing "Press Cmd+Z to undo", look for an Undo button instead
3. **Document the Limitation:** Record in findings that the step was skipped due to automation limits
4. **Continue Testing:** Don't let one limited step block the entire workflow
