# Selector Discovery Prompts

When exploring the codebase for multi-user workflows, use these search patterns.

## Persona-Specific Elements
```
Search: component name + persona role (e.g., "HostDashboard", "GuestView")
Look for: data-testid, role-based rendering, conditional displays
```

## Real-Time Elements
```
Search: "WebSocket" OR "SSE" OR "useSubscription" OR "onMessage"
Look for: Live counters, presence indicators, notification badges
```

## Shared State Elements
```
Search: "members" OR "participants" OR "viewers" OR "count"
Look for: data-testid on counter elements, dynamic text patterns
```

## Buttons
```
Search: "button" + "[text from workflow]"
Look for: data-testid, aria-label, className, onClick handler name
```

## Inputs
```
Search: "input" + "[field name]" OR "TextField" + "[label]"
Look for: name, id, placeholder, aria-label, data-testid
```
