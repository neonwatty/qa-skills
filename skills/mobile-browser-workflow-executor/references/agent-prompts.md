# Agent Prompts

Full prompts for agents delegated during workflow execution.

## Phase 4: UX Platform Evaluation Agent

```python
Task(
    task=f"""
Perform iOS Human Interface Guidelines (HIG) evaluation on mobile browser workflow: {workflow.name}

CONTEXT:
- Workflow URL: {workflow.url}
- Screenshots: workflows/screenshots/{workflow.slug}/
- Browser: Chrome mobile viewport (393x852)
- Engine: {engine}

EVALUATION CRITERIA:

1. NAVIGATION PATTERNS
   - Check for hamburger menus (anti-pattern)
   - Verify iOS-standard tab bar usage
   - Assess navigation bar implementation
   - Verify back button behavior

2. TOUCH TARGET SIZING
   - Measure all interactive elements
   - Minimum: 44x44 points (132x132 CSS pixels at 3x)
   - Check spacing between targets (min 8pt)
   - Verify thumb-reachable zones

3. NATIVE COMPONENT USAGE
   - Identify custom vs native-like controls
   - Check for iOS-standard gestures
   - Verify pull-to-refresh implementation
   - Assess modal presentation styles

4. VISUAL DESIGN
   - Color contrast ratios (WCAG AA minimum)
   - Typography scaling (Dynamic Type)
   - Spacing consistency (8pt grid)
   - Visual hierarchy clarity

5. PLATFORM CONVENTIONS
   - Destructive actions (red, confirmation)
   - Primary actions (blue, prominent)
   - Cancel patterns (top-left or bottom)
   - Alert/dialog styles

METHODOLOGY:
1. Load each screenshot from workflows/screenshots/{workflow.slug}/
2. Use browser_snapshot to analyze DOM structure
3. Use browser_evaluate to measure element dimensions
4. Analyze navigation structure

OUTPUT FORMAT:
Create findings in this structure:

## Finding: [Anti-Pattern Name]

**Severity**: Critical | High | Medium | Low
**Category**: Navigation | Touch Targets | Components | Visual | Platform
**Location**: [Specific element/screen]

**Current Implementation**: [Description]
**iOS HIG Violation**: [Specific guideline with reference]
**Impact**: [User experience, accessibility, platform issues]
**Recommended Solution**: [Specific iOS-native pattern]
**Implementation**: [Code examples - CSS/HTML]
**Visual Reference**: [Screenshot paths]

DELIVERABLE:
Save findings to .claude/plans/mobile-browser-workflow-findings.md
""",
    metadata={
        "phase": 4,
        "workflow": workflow.name,
        "delegation_reason": "Complex iOS HIG analysis requiring deep inspection"
    }
)
```

## iOS HIG Anti-Pattern Checklist

### Navigation Anti-Patterns
- **Hamburger Menu** - iOS Solution: Tab bar with 3-5 top-level sections (HIG > Navigation > Tab Bars)
- **Desktop-Style Top Nav** - iOS Solution: Navigation bar with hierarchical structure
- **Accordion Menus** - iOS Solution: Hierarchical push navigation (use sparingly)

### Touch Target Anti-Patterns
- **Small Buttons** (<44pt / 132px @ 3x) - iOS Solution: Minimum 44x44pt touch targets
- **Cramped Spacing** (<8pt apart) - iOS Solution: Minimum 8pt spacing between targets
- **Edge Targets** - iOS Solution: Place primary actions in thumb zone (bottom 60%)

### Component Anti-Patterns
- **Custom Select Dropdowns** - iOS Solution: Native picker or action sheet
- **Non-Standard Alerts** - iOS Solution: Native alert style with title + message + actions
- **Material Design Patterns** - iOS Solution: iOS-equivalent patterns

### Visual Anti-Patterns
- **Low Contrast** (<4.5:1) - iOS Solution: WCAG AA compliance (4.5:1 minimum)
- **Fixed Typography** (pixel-based) - iOS Solution: Relative units supporting Dynamic Type
- **Inconsistent Spacing** - iOS Solution: 8pt grid system

## Measurement Utilities

```javascript
const iOSHIGAudit = {
  measureTouchTargets: () => {
    const interactive = document.querySelectorAll('a, button, input, select, textarea, [role="button"], [role="link"], [onclick]');
    const results = [];
    interactive.forEach(el => {
      const rect = el.getBoundingClientRect();
      const dpr = window.devicePixelRatio || 3;
      const widthPt = rect.width / dpr;
      const heightPt = rect.height / dpr;
      results.push({
        element: el.tagName + (el.id ? '#' + el.id : '') + (el.className ? '.' + el.className.split(' ')[0] : ''),
        text: el.textContent.trim().substring(0, 30),
        widthPt: widthPt.toFixed(1),
        heightPt: heightPt.toFixed(1),
        meetsMinimum: widthPt >= 44 && heightPt >= 44,
        position: { top: rect.top, left: rect.left, inThumbZone: rect.top > (window.innerHeight * 0.4) }
      });
    });
    return results;
  },

  detectNavigationPattern: () => ({
    hamburger: !!document.querySelector('[aria-label*="menu" i], .hamburger, .menu-toggle, #menu-icon'),
    tabBar: !!document.querySelector('[role="tablist"], .tab-bar, nav[class*="bottom"]'),
    navBar: !!document.querySelector('header nav, [role="navigation"]'),
    accordion: !!document.querySelector('[aria-expanded], .accordion, details')
  }),

  checkColorContrast: () => {
    const textElements = document.querySelectorAll('p, span, a, button, h1, h2, h3, h4, h5, h6, li');
    const issues = [];
    textElements.forEach(el => {
      const computed = window.getComputedStyle(el);
      const color = computed.color;
      const bgColor = computed.backgroundColor;
      const textRGB = color.match(/\d+/g).map(Number);
      const bgRGB = bgColor.match(/\d+/g).map(Number);
      const textLuminance = (0.299 * textRGB[0] + 0.587 * textRGB[1] + 0.114 * textRGB[2]) / 255;
      const bgLuminance = (0.299 * bgRGB[0] + 0.587 * bgRGB[1] + 0.114 * bgRGB[2]) / 255;
      const contrast = Math.abs(textLuminance - bgLuminance);
      if (contrast < 0.5) {
        issues.push({
          element: el.tagName,
          text: el.textContent.trim().substring(0, 30),
          contrast: contrast.toFixed(2),
          color, bgColor
        });
      }
    });
    return issues;
  }
};
```

## Phase 8: Fix Mode Parallel Agents

```python
# Group fixes by independence
fix_groups = {
    "navigation": {
        "issues": ["hamburger-menu", "tab-bar-missing"],
        "files": ["components/header.html", "styles/navigation.css"]
    },
    "touch-targets": {
        "issues": ["small-buttons-product-card", "cramped-spacing-filters"],
        "files": ["styles/product-card.css", "styles/filters.css"]
    },
    "color-contrast": {
        "issues": ["low-contrast-footer", "low-contrast-secondary-text"],
        "files": ["styles/footer.css", "styles/typography.css"]
    },
    "components": {
        "issues": ["custom-dropdown", "non-standard-alerts"],
        "files": ["components/sort-dropdown.html", "scripts/alerts.js"]
    }
}

# For each group, spawn an agent with:
Task(
    task=f"""
Fix {group_name} issues in mobile browser workflow.

ISSUES TO FIX: {format_issues(group_data['issues'])}
FILES TO MODIFY: {group_data['files']}

CONTEXT:
- Mobile viewport: 393x852px (iPhone 14 Pro)
- Target platform: iOS web
- Design system: iOS HIG compliance

REQUIREMENTS:
1. Read current implementation from files
2. Apply fixes according to iOS HIG guidelines
3. Maintain existing functionality
4. Preserve surrounding code structure
5. Add comments explaining changes

REFERENCE:
See .claude/plans/mobile-browser-workflow-findings.md for detailed fix specifications.
""")
```

## Phase 9: Verification Agent

```python
Task(
    task=f"""
Verify that fixes applied in Phase 8 have resolved identified issues.

VERIFICATION WORKFLOW:
1. Re-execute all mobile browser workflows using Playwright MCP
2. Capture new screenshots in workflows/screenshots/{{workflow}}/fixed/
3. Measure touch targets to confirm 44pt minimum
4. Verify navigation pattern changed from hamburger to tab bar
5. Check color contrast ratios meet WCAG AA (4.5:1)
6. Confirm iOS HIG compliance for all modified components

PASS CRITERIA:
- All touch targets >= 44pt
- Tab bar navigation detected
- All contrast ratios >= 4.5:1
- Zero critical/high iOS HIG violations

DELIVERABLE:
Create verification report at .claude/plans/mobile-browser-workflow-verification.md
""")
```

## Phase 10-11: Report Generation Agents

### HTML Report Agent
Generate self-contained HTML with embedded CSS, device frame mockups, dark/light mode, interactive expand/collapse, and print-friendly styles. See [../examples/audit-report-template.html](../examples/audit-report-template.html) for the template.

### Markdown Report Agent
Generate GitHub-flavored Markdown with table of contents, collapsible sections, relative image links, and emoji status indicators.
