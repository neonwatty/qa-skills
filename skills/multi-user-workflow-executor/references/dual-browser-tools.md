# Dual-Browser MCP Tool Reference

This skill uses two browser automation tools simultaneously to simulate multi-user interaction:

- **Chrome MCP** = User A (primary/authenticated user, uses existing Chrome session)
- **Playwright MCP** = User B (secondary user, separate browser instance, auth via API or cookie injection)

## Capability Comparison

| Capability | Chrome MCP (User A) | Playwright MCP (User B) |
|---|---|---|
| Role | Primary/authenticated user | Secondary user |
| Session | User's existing Chrome session | Separate Playwright browser |
| Auth | Uses existing cookies/session | Sets up auth via API or cookie injection |
| Screenshots | `computer` action screenshot | `browser_snapshot` or screenshot tools |
| Navigation | `navigate` | `browser_navigate` |
| Element finding | `find` / `read_page` | `browser_snapshot` |
| Clicking | `computer` with `left_click` | `browser_click` |
| Text input | `computer` with `type` | `browser_fill_form` |

## Chrome MCP Tools (User A)

**Navigation:**
- `navigate({ url, tabId })` - Go to URL

**Finding Elements:**
- `find({ query, tabId })` - Natural language search, returns refs
- `read_page({ tabId, filter: 'interactive' })` - Get all interactive elements

**Interactions:**
- `computer({ action: 'left_click', coordinate: [x, y], tabId })`
- `computer({ action: 'left_click', ref: 'ref_1', tabId })` - Click by reference
- `computer({ action: 'type', text: '...', tabId })`
- `computer({ action: 'scroll', scroll_direction: 'down', coordinate: [x, y], tabId })`
- `computer({ action: 'left_click_drag', start_coordinate: [x1, y1], coordinate: [x2, y2], tabId })`
- `computer({ action: 'wait', duration: 2, tabId })`

**Screenshots:**
- `computer({ action: 'screenshot', tabId })` - Capture current state

**Inspection:**
- `get_page_text({ tabId })` - Extract text content
- `read_console_messages({ tabId, pattern: 'error' })` - Check for errors
- `read_network_requests({ tabId })` - Check API calls

**Forms:**
- `form_input({ ref, value, tabId })` - Set form field value

## Playwright MCP Tools (User B)

**Navigation:**
- `browser_navigate({ url })` - Navigate to URL

**Page State:**
- `browser_snapshot({})` - Get accessibility snapshot of current page

**Interactions:**
- `browser_click({ element, ref })` - Click an element by description or reference
- `browser_fill_form({ element, ref, value })` - Fill a form field
- `browser_select_option({ element, ref, values })` - Select dropdown option
- `browser_hover({ element, ref })` - Hover over an element
- `browser_drag({ startElement, endElement })` - Drag and drop

**Keyboard:**
- `browser_press_key({ key })` - Press a keyboard key
- `browser_type({ text, submit })` - Type text, optionally press Enter

**Tabs:**
- `browser_tab_list({})` - List open tabs
- `browser_tab_new({ url })` - Open new tab
- `browser_tab_select({ ref })` - Switch to tab

**Utilities:**
- `browser_wait({ time })` - Wait for specified milliseconds
- `browser_resize({ width, height })` - Resize viewport
- `browser_handle_dialog({ accept, promptText })` - Handle alert/confirm/prompt dialogs
- `browser_file_upload({ paths })` - Upload files
- `browser_pdf_save({})` - Save page as PDF
- `browser_close({})` - Close the browser

## Step Routing

Route workflow steps to the correct browser based on persona prefix:

**[User A] steps -> Chrome MCP tools:**
- "Navigate to [URL]" -> `navigate`
- "Click [element]" -> `find` to locate, then `computer` with `left_click`
- "Type [text]" -> `computer` with `type` action
- "Verify [condition]" -> `read_page` or `get_page_text` to check
- "Drag [element]" -> `computer` with `left_click_drag`
- "Scroll [direction]" -> `computer` with `scroll`
- "Wait [seconds]" -> `computer` with `wait`

**[User B] steps -> Playwright MCP tools:**
- "Navigate to [URL]" -> `browser_navigate`
- "Click [element]" -> `browser_click` with element description or coordinates
- "Type [text]" -> `browser_fill_form` with field selector and value
- "Verify [condition]" -> `browser_snapshot` to inspect page state
- "Wait [seconds]" -> wait via appropriate delay
