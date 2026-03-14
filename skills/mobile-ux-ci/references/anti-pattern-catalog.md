# Anti-Pattern Detection Catalog

## Navigation Anti-Patterns

| Anti-Pattern | Why It's Wrong | What to Test |
|--------------|---------------|--------------|
| Hamburger menu | iOS uses tab bars | `.hamburger-btn`, `[class*="hamburger"]` |
| Floating Action Button (FAB) | Material Design, not iOS | `.fab`, `[class*="floating-action"]` |
| Breadcrumb navigation | iOS uses back button | `.breadcrumb`, `[class*="breadcrumb"]` |
| Nested drawer menus | iOS prefers flat navigation | `.drawer`, `[class*="drawer"]` |

## Touch Target Issues

| Issue | Standard | What to Test |
|-------|----------|--------------|
| Small buttons | iOS: 44x44pt, WCAG: 24x24px | `boundingBox()` on all `button, a, [role="button"]` |
| Targets too close | 8px minimum spacing | Measure distance between interactive elements |

## Component Anti-Patterns

| Anti-Pattern | iOS Alternative | What to Test |
|--------------|-----------------|--------------|
| Native `<select>` | iOS picker wheels | `select:visible` count |
| Checkboxes | Toggle switches | `input[type="checkbox"]` count |
| Material snackbars | iOS alerts/banners | `.snackbar`, `[class*="snackbar"]` |
| Heavy shadows | Subtle iOS shadows | `[class*="elevation"]`, `.shadow-xl` |

## Layout Issues

| Issue | What to Test |
|-------|--------------|
| Horizontal overflow | `body.scrollWidth > html.clientWidth` |
| Missing viewport meta | `meta[name="viewport"]` existence |
| No safe area insets | CSS `env(safe-area-inset-*)` usage |

## Text & Selection

| Issue | What to Test |
|-------|--------------|
| UI text selectable | `user-select` CSS property |
| Font too small | Font sizes below 14px |

## Interaction Issues

| Issue | What to Test |
|-------|--------------|
| Hover-dependent UI | Elements with opacity:0 and hover classes |
| Double-tap zoom | `touch-action: manipulation` |
| Canvas gesture conflicts | `touch-action: none` on canvas |

---

## Severity Reference

### Critical (Should Always Fail CI)

1. **Hamburger Menu for Primary Navigation**
   - Why: iOS users expect tab bars at bottom
   - Reference: [iOS vs Android Navigation](https://www.learnui.design/blog/ios-vs-android-app-ui-design-complete-guide.html)

2. **Floating Action Button (FAB)**
   - Why: Material Design pattern, not iOS
   - Reference: [Material vs iOS](https://medium.com/@helenastening/material-design-v-s-ios-11-b4f87857814a)

3. **Touch Targets < 44pt**
   - Why: Apple HIG requirement for accessibility
   - Reference: [Apple HIG](https://developer.apple.com/design/human-interface-guidelines/)

4. **Horizontal Overflow**
   - Why: Content should fit viewport on mobile
   - Reference: Basic responsive design

### Warning (Should Log but May Not Fail)

1. **Native `<select>` Elements**
   - Why: iOS apps use picker wheels
   - Note: Some selects are acceptable for accessibility

2. **Checkboxes**
   - Why: iOS uses toggle switches
   - Note: Checkmarks in lists are acceptable

3. **Text Selection on UI**
   - Why: Native apps prevent selecting UI text
   - Note: Content text should remain selectable

4. **No Safe Area Insets**
   - Why: Content may go under notch
   - Note: Only relevant for notched devices

### Informational (Suggestions Only)

1. **Heavy Shadows (Material Elevation)**
2. **Missing touch-action: manipulation**
3. **Non-system fonts**

## Sources

- [Apple Human Interface Guidelines](https://developer.apple.com/design/human-interface-guidelines/)
- [iOS vs Android Design](https://www.learnui.design/blog/ios-vs-android-app-ui-design-complete-guide.html)
- [Touch Target Sizes](https://www.smashingmagazine.com/2023/04/accessible-tap-target-sizes-rage-taps-clicks/)
- [PWA Native Feel](https://www.netguru.com/blog/pwa-ios)
- [WCAG 2.5.8 Target Size](https://www.w3.org/WAI/WCAG21/Understanding/target-size.html)
