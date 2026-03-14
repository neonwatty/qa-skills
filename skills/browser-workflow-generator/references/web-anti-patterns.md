# Browser/Web Platform UX Anti-Patterns

When generating workflows, include UX verification steps that check for these common anti-patterns where web apps incorrectly use native mobile conventions or miss web-specific requirements.

## Navigation Anti-Patterns

| Anti-Pattern | Web Convention | What to Check |
|--------------|----------------|---------------|
| Gesture-only navigation | Click + gesture alternatives | All navigation should work with mouse clicks |
| Breaking back button | URL-based navigation | Browser back button should work intuitively |
| No URL for states | Deep linkable URLs | Important states should have shareable URLs |
| Tab bar at bottom | Top navigation or sidebar | Bottom tab bars are mobile patterns, not web |
| Swipe-only carousels | Arrow buttons + indicators | Carousels need visible click controls |

## Interaction Anti-Patterns

| Anti-Pattern | Web Convention | What to Check |
|--------------|----------------|---------------|
| Missing hover states | Clear hover feedback | All interactive elements need hover indication |
| No focus indicators | Visible focus rings | Keyboard navigation needs visible focus states |
| Touch-sized-only buttons | Standard web button sizes | Buttons can be smaller than 44pt on web |
| Pull-to-refresh | Refresh button or auto-refresh | Web doesn't support native pull-to-refresh |
| Long press menus | Right-click or visible menu buttons | Context menus should use right-click on web |

## Visual Anti-Patterns

| Anti-Pattern | Web Convention | What to Check |
|--------------|----------------|---------------|
| Full-screen modals everywhere | Inline expansion or sized modals | Web modals typically don't need full-screen |
| iOS/Android specific styling | Platform-agnostic web design | Avoid native mobile component styling |
| No responsive breakpoints | Responsive design | Layout should adapt to viewport width |
| Mobile-only viewport | Desktop-first or responsive | Should work well at 1200px+ widths |
| App-like splash screens | Immediate content loading | Web should show content quickly, not splash |

## Component Anti-Patterns

| Anti-Pattern | Web Convention | What to Check |
|--------------|----------------|---------------|
| Native mobile pickers | HTML select or custom dropdowns | Use web-native form components |
| Action sheets sliding up | Dropdown menus or modals | Use web-appropriate menu patterns |
| iOS-style toggle switches | Checkboxes or web-styled toggles | Consider web conventions for boolean inputs |
| Floating bottom bars | Fixed headers or inline CTAs | Sticky bottom bars can feel app-like |
| Edge swipe gestures | Visible navigation buttons | Don't rely on edge swipes for critical actions |

## Accessibility Anti-Patterns

| Anti-Pattern | Web Convention | What to Check |
|--------------|----------------|---------------|
| No keyboard navigation | Full keyboard support | All features accessible via keyboard |
| Missing ARIA labels | Proper accessibility markup | Screen readers should understand the UI |
| Color-only indicators | Color + icon/text indicators | Don't rely solely on color for meaning |
| Auto-playing media | User-initiated playback | Media should not auto-play with sound |
| Trapped focus in modals | Proper focus management | Focus should be trapped correctly in modals |

## Workflow UX Verification Steps

When writing workflows, include verification steps for platform appropriateness:

```markdown
## Workflow: [Name]

...

6. Verify web platform conventions
   - Verify all interactive elements have hover states
   - Verify browser back button works correctly
   - Verify keyboard navigation is possible
   - Verify layout is responsive (try different viewport widths)
   - Verify URLs are shareable/deep-linkable for important states
```
