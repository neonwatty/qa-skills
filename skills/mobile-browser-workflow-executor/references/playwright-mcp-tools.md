# Playwright MCP Tool Reference

## Playwright MCP Tools (Primary Engine)

| Tool | Purpose | Mobile Usage |
|------|---------|--------------|
| `browser_resize` | Set viewport size | Set to 393x852 (iPhone 14 Pro) |
| `browser_navigate` | Load URL | Navigate to workflow start |
| `browser_snapshot` | Get accessibility tree | Discover elements for interaction |
| `browser_take_screenshot` | Capture visual state | Document before/after states |
| `browser_click` | Click element | Simulate tap interaction |
| `browser_type` | Enter text | Simulate keyboard input |
| `browser_evaluate` | Run JavaScript | Measure elements, override UA, inject utilities |
| `browser_wait_for` | Wait for condition | Wait for elements or time |
| `browser_fill_form` | Fill multiple fields | Batch form completion |
| `browser_select_option` | Select dropdown option | Choose from native select |
| `browser_press_key` | Press keyboard key | Submit forms, navigate |
| `browser_swipe` | Scroll gesture | Mobile-specific scrolling |

## Claude-in-Chrome Tools (Alternative Engine)

| Tool | Purpose | Mobile Usage |
|------|---------|--------------|
| `tabs_context_mcp` | Get tab context | Initialize session |
| `tabs_create_mcp` | Create new tab | Start workflow in fresh tab |
| `resize_window` | Resize browser | Best-effort mobile viewport |
| `navigate` | Load URL | Navigate to workflow start |
| `read_page` | Get accessibility tree | Discover elements |
| `find` | Search for elements | Natural language element discovery |
| `computer` (screenshot) | Capture screen | Document visual state |
| `computer` (left_click) | Click element | Simulate tap |
| `computer` (type) | Enter text | Keyboard input |
| `computer` (scroll) | Scroll page | Simulate swipe |
| `javascript_tool` | Execute JS | Override UA, measure elements |
| `form_input` | Set form values | Batch form completion |

## Tool Selection Strategy

**Use Playwright MCP when**:
- Starting fresh session
- Need precise viewport control
- Require mobile-specific emulation
- Want programmatic element selection

**Use Claude-in-Chrome when**:
- User prefers existing browser
- Need to interact with authenticated sessions
- Want visual debugging in real browser
- Viewport precision less critical

## Action Mapping (Playwright MCP)

| Workflow Action | Playwright Tool | Implementation |
|----------------|-----------------|----------------|
| Navigate | `browser_navigate` | `browser_navigate({ url: TARGET })` |
| Tap | `browser_click` | `browser_click({ ref: REF })` with touch event |
| Type | `browser_type` | `browser_type({ ref: REF, text: VALUE })` |
| Swipe Up | `browser_evaluate` | Custom scroll with touch simulation |
| Swipe Down | `browser_evaluate` | Custom scroll with touch simulation |
| Wait | `browser_wait_for` | `browser_wait_for({ text: EXPECTED })` |
| Verify | `browser_snapshot` | Assert element presence |
| Screenshot | `browser_take_screenshot` | Capture visual state |

## Step Execution Template

```python
# Before step execution
await browser_snapshot()  # Get current page state
await browser_take_screenshot({
    filename: f"workflows/screenshots/{workflow_name}/before-step-{step_num}.png",
    type: "png"
})

# Execute step based on action type
if action == "Tap":
    snapshot = await browser_snapshot()
    target_ref = identify_element(snapshot, step.target)
    await browser_click({ element: step.target, ref: target_ref })

elif action == "Type":
    snapshot = await browser_snapshot()
    target_ref = identify_element(snapshot, step.target)
    await browser_type({
        element: step.target,
        ref: target_ref,
        text: step.value,
        slowly: True
    })

elif action == "Swipe":
    direction = step.direction
    await browser_evaluate({
        function: f`() => {{
            const scrollDistance = {400 if direction in ['up', 'down'] else 300};
            if ('{direction}' === 'up') window.scrollBy({{ top: scrollDistance, behavior: 'smooth' }});
            else if ('{direction}' === 'down') window.scrollBy({{ top: -scrollDistance, behavior: 'smooth' }});
            else if ('{direction}' === 'left') window.scrollBy({{ left: scrollDistance, behavior: 'smooth' }});
            else if ('{direction}' === 'right') window.scrollBy({{ left: -scrollDistance, behavior: 'smooth' }});
        }}`
    })

elif action == "Verify":
    snapshot = await browser_snapshot()
    element_found = verify_element_presence(snapshot, step.target)
    if not element_found:
        raise WorkflowError(f"Verification failed: {step.expected}")

# After step execution
await browser_wait_for({ time: 1 })
await browser_take_screenshot({
    filename: f"workflows/screenshots/{workflow_name}/after-step-{step_num}.png",
    type: "png"
})
```

## Touch Event Simulation (Advanced)

For scenarios requiring native-like touch events:

```javascript
await browser_evaluate({
  function: `(element) => {
    const rect = element.getBoundingClientRect();
    const touchObj = new Touch({
      identifier: Date.now(),
      target: element,
      clientX: rect.left + rect.width / 2,
      clientY: rect.top + rect.height / 2,
      radiusX: 2.5,
      radiusY: 2.5,
      rotationAngle: 0,
      force: 1
    });

    const touchEvent = new TouchEvent('touchstart', {
      cancelable: true,
      bubbles: true,
      touches: [touchObj],
      targetTouches: [touchObj],
      changedTouches: [touchObj]
    });

    element.dispatchEvent(touchEvent);
  }`,
  ref: target_ref,
  element: step.target
});
```

## Playwright MCP Initialization

```javascript
// 1. Resize viewport to mobile dimensions
await browser_resize({ width: 393, height: 852 });

// 2. Override user agent via evaluate
await browser_evaluate({
  function: `() => {
    Object.defineProperty(navigator, 'userAgent', {
      get: () => 'Mozilla/5.0 (iPhone; CPU iPhone OS 17_0 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/17.0 Mobile/15E148 Safari/604.1'
    });
  }`
});

// 3. Set mobile viewport meta tag emulation
await browser_evaluate({
  function: `() => {
    const meta = document.createElement('meta');
    meta.name = 'viewport';
    meta.content = 'width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no';
    document.head.appendChild(meta);
  }`
});

// 4. Enable touch events
await browser_evaluate({
  function: `() => {
    document.documentElement.style.touchAction = 'manipulation';
  }`
});
```

## Claude-in-Chrome Initialization

```javascript
// 1. Get tab context
await tabs_context_mcp({ createIfEmpty: true });

// 2. Resize window (best effort)
await resize_window({ tabId: TAB_ID, width: 393, height: 852 });

// 3. Override user agent
await javascript_tool({
  tabId: TAB_ID,
  action: 'javascript_exec',
  text: `
    Object.defineProperty(navigator, 'userAgent', {
      get: () => 'Mozilla/5.0 (iPhone; CPU iPhone OS 17_0 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/17.0 Mobile/15E148 Safari/604.1'
    });
  `
});
```
