# Chrome MCP Tool Reference

## Navigation

- `navigate({ url, tabId })` - Go to URL

## Finding Elements

- `find({ query, tabId })` - Natural language search, returns refs
- `read_page({ tabId, filter: 'interactive' })` - Get all interactive elements

## Interactions

- `computer({ action: 'left_click', coordinate: [x, y], tabId })` - Click at coordinates
- `computer({ action: 'left_click', ref: 'ref_1', tabId })` - Click by reference
- `computer({ action: 'type', text: '...', tabId })` - Type text
- `computer({ action: 'scroll', scroll_direction: 'down', coordinate: [x, y], tabId })` - Scroll
- `computer({ action: 'left_click_drag', start_coordinate: [x1, y1], coordinate: [x2, y2], tabId })` - Drag and drop
- `computer({ action: 'wait', duration: 2, tabId })` - Wait N seconds

## Screenshots

- `computer({ action: 'screenshot', tabId })` - Capture current state

## Inspection

- `get_page_text({ tabId })` - Extract text content
- `read_console_messages({ tabId, pattern: 'error' })` - Check for errors
- `read_network_requests({ tabId })` - Check API calls

## Forms

- `form_input({ ref, value, tabId })` - Set form field value
