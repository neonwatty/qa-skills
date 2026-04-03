# Measurement Reliability Improvements Spec

**Date:** 2026-04-02
**Status:** Research complete, ready for implementation planning
**Parent spec:** `2026-04-02-performance-mobile-expansion-design.md`

## Summary

Four independent critique agents and two conversion agents analyzed the qa-skills measurement stack and produced actionable improvements. This document captures all findings for implementation.

## Check Classification: Before and After

### Before conversion
| Tier | Count | % |
|------|-------|---|
| DETERMINISTIC | 33 | 33% |
| HEURISTIC | 40 | 40% |
| LLM-JUDGMENT | 26 | 27% |
| **Total** | **99** | |

### After conversion
| Tier | Count | % |
|------|-------|---|
| DETERMINISTIC | 58 | 59% |
| HEURISTIC (reliable, <5% error) | 28 | 28% |
| HEURISTIC (needs Playwright interaction) | 8 | 8% |
| LLM-JUDGMENT (with pre-filter, 75-85% reduction) | 5 | 5% |
| Pure LLM-JUDGMENT | 0 | 0% |
| **Total** | **99** | |

## Priority 1: Structural Changes

### 1a. Tier every check as [D] / [H] / [J]

Tag every check in all three reference files (ux-auditor.md, mobile-ux-auditor.md, performance-profiler.md) with its measurement tier. Report separate sub-scores:

```
Deterministic: 52/58 (90%)
Heuristic: 22/28 (79%)
Interaction: 6/8 (75%)
LLM-Assisted: 4/5 (80%)
Overall: 84/99 (85%)
```

### 1b. Compound conditions (10 checks)

Prevent scorecard gaming by requiring multiple conditions:

1. `autocomplete` — must be a valid HTML token (not "on"/"off"/empty)
2. `prefers-reduced-motion` — rule body must modify animation/transition properties
3. Touch target >= 44x44 — AND visible content area >= 30x30
4. Pagination present — AND has > 1 navigable page
5. Form label exists — AND label is visible AND positioned near field
6. Error message specific — AND text includes field label AND contains instruction
7. Hover state present — AND perceivable style change (not 1px subtle shift)
8. Skeleton screen present — AND element has visible dimensions AND has animation
9. Empty state has CTA — AND button actually navigates somewhere
10. Heading frequency — AND headings are semantically varied (not all identical)

### 1c. Graduated scoring (7 checks)

Replace binary pass/fail with 0 / 0.5 / 1.0 for cliff-effect checks:

| Check | 0 | 0.5 | 1.0 |
|-------|---|-----|-----|
| Touch target size | < 24px | 24-43px | >= 44px |
| Color contrast | < 2:1 | 2:1-4.4:1 | >= 4.5:1 |
| Scroll depth ratio | > 8x | 3-8x | <= 3x |
| Font sizes count | > 12 | 7-12 | <= 6 |
| Information density | > 700 words | 300-700 | 150-300 |
| Animation duration | > 1500ms | 600-1500ms | 100-600ms |
| Line length | > 100 or < 20 | 75-100 or 20-30 | 30-75 |

### 1d. Category weighting

| Weight | Categories |
|--------|-----------|
| 2x | Accessibility, Touch & Interaction |
| 1.5x | Forms, Typography (contains contrast), iOS Safari, Interstitials |
| 1x | Navigation, Data Display, Feedback, Viewport |
| 0.5x | Visual Complexity, Animation & Motion, Gestures |

### 1e. Critical floor rule

If any CRITICAL-severity check fails, total score capped at 50%.

## Priority 2: Script Reliability Fixes

### 2a. Contrast ratio computation
- Parse colors numerically (not string-based transparency check)
- Handle alpha channel blending
- Flag `background-image` as indeterminate (not false-pass)
- Add `button, input, textarea, select, figcaption, blockquote` to selector

### 2b. CLS session windows
- Implement 1s gap / 5s max window algorithm per web.dev spec
- Current sum-all approach overcounts vs industry tools

### 2c. Long Tasks / TBT
- `longtask` doesn't reliably support `buffered: true`
- Inject observer before navigation via `page.addInitScript()`
- Add `try/catch` and `{ available: false }` for non-Chromium

### 2d. Whitespace / visual balance double-counting
- Use pixel-sampling via `elementFromPoint()` for whitespace (20px grid)
- Use leaf-only + viewport-clipped Ngo formula for balance

### 2e. Form keyword matching
- Change `combined.includes('tel')` to `/\btel\b/i.test(combined)`

### 2f. Page-settled detection
- Wait for `networkidle` + `document.fonts.ready` + no running animations before measurements

### 2g. Shadow DOM traversal
- Add recursive `shadowRoot` traversal utility to all DOM queries

### 2h. Sub-pixel rounding
- Use `Math.round(val) % 4 === 0` instead of `val % 4 === 0` for spacing grid

### 2i. Navigation Timing guard
- Check `nav.loadEventEnd === 0` before computing load_complete_ms

### 2j. Font audit leaf-only
- Filter to leaf text nodes only (no container double-counting)
- Wait for `document.fonts.ready` before measuring

### 2k. Repeated item detection
- Extend beyond `ul/ol/table` to detect div-grid patterns via child-class analysis

### 2l. Alignment clustering
- Filter to block-level elements only (exclude inline spans/links)
- Use cluster-start tracking (not adjacent-value comparison)

## Priority 3: Missing Metrics

### 3a. WCAG 2.2 AA criteria (6 new checks for UX-auditor)
- Focus Not Obscured (2.4.11)
- Dragging Movements (2.5.7)
- Target Size on desktop (2.5.8)
- Consistent Help (3.2.6)
- Redundant Entry (3.3.7)
- Accessible Authentication (3.3.8)

### 3b. Dark mode contrast testing
- Run contrast checks twice via `page.emulateMedia({ colorScheme: 'dark' })`

### 3c. Security headers (adversarial agent)
- CSP, HSTS, SRI, Permissions-Policy, cookie flags, X-Frame-Options, Referrer-Policy

### 3d. Multiple-run measurement
- Minimum 3 runs, report median for timing metrics
- Flag changes < 10% as "within noise"

### 3e. SPA soft navigation gap
- Document limitation
- Add link-click measurement mode for client-side routing

### 3f. INP unmeasurable in lab
- Document TBT as proxy explicitly

### 3g. Font loading strategy checks (performance agent)
- `font-display: swap/optional`
- Preloading web fonts
- `size-adjust` for fallbacks

### 3h. Resource hint checks (performance agent)
- `preconnect`, `dns-prefetch`, `preload`, `fetchpriority="high"` on LCP images

## Priority 4: Converted Check Scripts

### 24 Heuristic checks converted to Deterministic

Full JavaScript implementations available from the heuristic-to-deterministic agent. Key conversions:

1. Spacing grid: `Math.round(val) % 4 === 0` (sub-pixel fix)
2. Color off-by-one: Delta-E CIELAB clustering (JND < 3.0)
3. Border radii: Count distinct patterns
4. Shadows: Normalize + count distinct values
5. Disabled state: 4-signal check (opacity, pointer-events, cursor, color)
6. Tab order: DOM order vs visual position comparison
7. Text truncation: `scrollWidth > clientWidth` without ellipsis
8. Image constraints: width/height attrs + container overflow
9. Current location: aria-current + active class + style diff
10. CTA count: Exclude nav/footer, classify by primary styling
11. Autofill: autocomplete attribute validation against HTML spec tokens
12. Multi-step progress: Step indicator DOM detection
13. Error position: Distance measurement between error and field
14. Destructive verb: Keyword matching on button text
15. Skeleton screen: Class + CSS keyframe detection
16. Pull-to-refresh: PTR class + overscroll-behavior
17. Repeated items: Child-class pattern analysis
18. Pagination: Element + item count check
19. Filter controls: Filter-related element queries
20. Virtual scroll: Library class + spacer + translateY detection
21. Visual balance: Leaf-only + viewport-clipped Ngo formula
22. Whitespace ratio: Pixel-sampling grid via `elementFromPoint()`
23. Button variations: 6-property normalized fingerprint counting
24. Heading frequency: TreeWalker with script/style exclusion

### 20 LLM-Judgment checks converted to Heuristic

Full JavaScript implementations available from the LLM-judgment-to-deterministic agent. Key conversions:

1. Icon style (outline/filled): SVG stroke/fill geometry analysis (85% confidence)
2. Error messages specific: Known-bad pattern dictionary (90% confidence)
3. Button labels action-oriented: Verb dictionary matching (85% confidence)
4. Placeholder text quality: Label-vs-example detection (85% confidence)
5. Confirmation messages: Generic-pattern + subject-mention check (80%)
6. Empty state quality: 3-component structural check (95% confidence)
7. Content not color-only: Link underline + form border + status indicator detection (80%)
8. Dead ends: Outbound link/button/nav count (90% confidence)
9. Error actionability: Field-name + fix-guidance + proximity (85%)
10. iOS native feel: MDC class + checkbox shape + FAB geometry + hamburger + toast detection (90%)

### 5 checks remain LLM-Judgment with pre-filters (75-85% reduction)

1. Default state clarity: toggle/radio/select pre-filter
2. Tooltips present where needed: icon-only button + truncated text pre-filter
3. Z-index stacking correct: z-index war + overlay-under-content pre-filter
4. Optimistic updates: toggle/like/form/delete candidate identification
5. Gesture support: swipe blocker + PTR + library detection pre-filter

## Conflicting Check Pairs (9)

1. Whitespace ratio vs scroll depth
2. Touch target size vs tap spacing vs nav item count
3. CTA count = 1 vs empty state needs CTA
4. Information density vs heading frequency
5. Visible fields per section vs single-column form layout
6. Body text >= 16px vs line length 30-50 chars on mobile
7. Overlay coverage < 30% vs popup close button >= 44x44
8. Font sizes <= 6 vs responsive type scales (sub-pixel rounding)
9. Colors <= 15 vs data visualization pages

## Threshold Citation Status

### Research-backed (no changes needed)
- LCP/CLS/INP/FCP/TTFB thresholds (Google Web Vitals)
- Touch targets 44pt (Apple HIG), 48dp (MD3), 24px (WCAG 2.5.8)
- Contrast ratios (WCAG 2.1)
- Line height 1.5x (WCAG 1.4.12)
- Content line length 45-75 chars (Bringhurst, Baymard)
- Flesch-Kincaid 7-8th grade (US government standard)
- MD3 elevation levels, duration tokens, easing curves

### Industry convention (acceptable, flag as [convention])
- Font sizes <= 6, font families <= 2 (typography best practice)
- Button variations <= 3 (design system convention)
- Nav items 5-7 (Miller's Law derived)
- Animation duration 200-300ms (NNG + MD3)
- CTA count = 1 (conversion optimization)

### Uncited heuristics (flag as [heuristic])
- Whitespace ratio 30-50%
- Visual balance Ngo > 0.85
- Information density 150-300 words/viewport
- Heading frequency every 200-300 words
- Alignment clusters <= 5
- Scroll depth <= 3x
- Overlay coverage < 30%
- Sticky banner < 15%
- Tap spacing >= 8px
- HTTP requests <= 50 (outdated for HTTP/2)
