# Known Automation Limitations

## Chrome MCP Limitations (User A)

1. **Keyboard Shortcuts**
   - System-level shortcuts (Cmd+Z, Cmd+C, Cmd+V, etc.) may cause extension disconnection
   - Browser shortcuts that trigger native behavior can interrupt the session
   - **Workaround:** Use UI buttons instead of keyboard shortcuts when available

2. **Native Browser Dialogs**
   - `alert()`, `confirm()`, `prompt()` dialogs block all browser events
   - File upload dialogs (OS-level file picker)
   - Print dialogs
   - **Workaround:** Skip steps requiring these, or flag for manual testing

3. **Pop-ups and New Windows**
   - Pop-ups that open in new windows outside the MCP tab group
   - OAuth flows that redirect to external authentication pages
   - **Workaround:** Document as requiring manual verification

4. **System-Level Interactions**
   - Browser permission prompts (camera, microphone, notifications, location)
   - Download dialogs and download management
   - Browser settings and preferences pages
   - **Workaround:** Pre-configure permissions or skip these steps

## Playwright MCP Limitations (User B)

1. **No Existing Session**
   - Playwright runs a fresh browser instance with no existing cookies/sessions
   - Authentication must be set up explicitly (login flow, cookie injection, or API tokens)
   - **Workaround:** Include auth setup steps in Phase 2

2. **Separate Browser Context**
   - Playwright runs Chromium, not the user's actual Chrome browser
   - Extensions, saved passwords, and browser profiles are not available
   - **Workaround:** Use API-based auth or explicit login flows

3. **Screenshot Format Differences**
   - Playwright screenshots may differ in format/resolution from Chrome MCP
   - Side-by-side comparisons may need normalization
   - **Workaround:** Note format differences in reports

4. **Network Isolation**
   - Playwright browser has its own network stack
   - Cookies set in Chrome are not shared with Playwright
   - **Workaround:** Set up auth independently in each browser

5. **Dialog Handling**
   - Playwright can handle dialogs programmatically via `browser_handle_dialog`
   - But dialogs must be handled before they appear (pre-register handler)
   - **Workaround:** Set up dialog handlers before triggering dialog-producing actions

## Cross-Browser Limitations

1. **Timing Coordination**
   - No built-in synchronization between Chrome MCP and Playwright MCP
   - Steps are executed sequentially, not truly simultaneously
   - **Workaround:** Use polling with timeouts for sync assertions

2. **State Isolation**
   - Changes in one browser are only visible in the other through the application's sync mechanism
   - Direct DOM/state sharing between browsers is not possible
   - **Workaround:** Rely on application-level sync (WebSocket, polling, SSE)

3. **Screenshot Timing**
   - Screenshots from both browsers are taken sequentially, not simultaneously
   - Small timing differences may exist between User A and User B screenshots
   - **Workaround:** Add short waits before cross-browser screenshots

## Handling Limited Steps

When a workflow step involves a known limitation:

1. **Mark as [MANUAL]:** Note the step requires manual verification
2. **Try Alternative:** If testing keyboard shortcuts, look for UI buttons instead
3. **Document the Limitation:** Record in findings that the step was skipped due to automation limits
4. **Continue Testing:** Don't let one limited step block the entire workflow
