# Native iOS Feel Requirements

Since these web apps will become PWAs or wrapped apps, they must feel **native to iOS**.

## Navigation (must feel like native iOS)

- Use tab bars for primary navigation, not hamburger menus
- Navigation should push/pop like native UINavigationController
- Back gestures should work naturally (or provide back button)
- Modals should slide up from bottom like native sheets

## Touch & Interaction

- All tap targets must be at least 44x44pt
- Consider thumb reach zones for primary actions
- Animations should feel native (spring physics, not CSS ease-in-out)
- Provide appropriate visual feedback for taps
- Haptic feedback patterns where appropriate

## Components (should match native iOS)

- Use iOS-style pickers, not web dropdowns
- Toggle switches, not checkboxes
- iOS-style action sheets, not Material Design
- Native-feeling form inputs

## Visual Design

- Follow iOS Human Interface Guidelines typography
- Subtle shadows and rounded corners (not Material elevation)
- SF Pro or system font stack
- iOS color semantics (system colors, semantic backgrounds)

## Device Considerations

- Safe area insets on notched devices
- Keyboard avoidance for forms
- Support both portrait and landscape if appropriate
- Test responsive behavior across different screen sizes

## Platform-Specific Notes

### iOS Simulator (Safari)
- Test on different iPhone screen sizes (SE, standard, Pro Max)
- Haptic feedback can be verified through simulator logs
- Native Safari rendering provides the most accurate component appearance

### Mobile Browser (Chrome Viewport)
- Test responsive behavior at 393x852 viewport (iPhone 15 Pro)
- Chrome DevTools mobile emulation does not support haptic feedback testing
- Focus on visual layout and interaction sizing rather than native rendering fidelity
