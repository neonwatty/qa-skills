# iOS Platform UX Anti-Patterns

Since the goal is a **native iOS feel**, check for these anti-patterns that make web apps feel like web apps instead of native iOS apps.

## Navigation Anti-Patterns

| Anti-Pattern | iOS Convention | What to Check |
|--------------|----------------|---------------|
| Hamburger menu (three lines) | Tab bar at bottom | Primary navigation should use tab bar, not hidden drawer |
| Floating Action Button (FAB) | Navigation bar buttons | Primary actions belong in top-right nav bar, not floating circle |
| Breadcrumb navigation | Back button + title | iOS uses single back button with previous screen title |
| Bottom sheets for navigation | Modal presentations or push | Navigation should push onto stack, not slide up sheets |
| Nested hamburger menus | Flat tab structure | iOS prefers flat hierarchy with tabs, not deep menu nesting |

## Interaction Anti-Patterns

| Anti-Pattern | iOS Convention | What to Check |
|--------------|----------------|---------------|
| Tiny tap targets (<44pt) | Minimum 44x44pt touch targets | All interactive elements should be easily tappable |
| Text-only buttons | Styled buttons or icons | Primary actions should have clear button styling |
| Swipe-only actions | Swipe + visible alternative | Critical actions need visible UI, not just swipe gestures |
| Long press as primary action | Long press for secondary | Long press should reveal options, not be required |
| Pull-to-refresh everywhere | Only in scrollable lists | Pull-to-refresh is for list content, not all screens |

## Visual Anti-Patterns

| Anti-Pattern | iOS Convention | What to Check |
|--------------|----------------|---------------|
| Custom form controls | Native UIKit/SwiftUI appearance | Use iOS-styled Picker, DatePicker, Toggle, not custom widgets |
| Web-style dropdowns | iOS Picker wheels or menus | Dropdowns should use native picker presentation |
| Dense information layout | Generous spacing and hierarchy | iOS favors readability over density |
| Material Design styling | iOS Human Interface Guidelines | Avoid Android-specific visual patterns |
| Fixed headers that cover content | iOS navigation bar behavior | Headers should integrate with iOS navigation system |

## Component Anti-Patterns

| Anti-Pattern | iOS Convention | What to Check |
|--------------|----------------|---------------|
| Toast notifications | iOS alerts or banners | Use native alert styles, not Android-style toasts |
| Snackbars | Action sheets or alerts | Bottom notifications should follow iOS patterns |
| Cards with heavy shadows | Subtle iOS card styling | iOS uses subtle shadows and rounded corners |
| Outlined text fields | iOS text field styling | Text fields should match iOS native appearance |
| Checkboxes | iOS Toggle switches or checkmarks | Use SF Symbols checkmarks or Toggle for boolean states |

## Workflow UX Verification Steps

When writing workflows, include verification steps for platform appropriateness:

```markdown
## Workflow: [Name]

...

6. Verify iOS platform conventions
   - Verify primary navigation uses tab bar (not hamburger menu)
   - Verify interactive elements are at least 44x44pt
   - Verify forms use iOS-style components (Picker, Toggle, etc.)
   - Verify navigation follows iOS back-button pattern
   - Verify visual styling follows iOS Human Interface Guidelines
   - Verify content respects safe areas (top 44pt, bottom 34pt)
   - Verify animations feel native (spring physics)
```

## Platform-Specific Notes

### iOS Simulator (Safari)
- Test with actual iOS Simulator for most accurate native component rendering
- UIKit/SwiftUI component references apply directly to native wrapper scenarios (Capacitor, Tauri)
- SF Symbols and system fonts render natively in Safari on iOS

### Mobile Browser (Chrome Viewport)
- Chrome viewport emulation approximates iOS rendering but does not use native components
- Focus on layout, spacing, touch targets, and navigation patterns rather than pixel-perfect native components
- Verify content respects iPhone 15 Pro safe areas at 393x852 viewport
