# QA-Skills Plugin: Performance & Mobile Expansion

**Date:** 2026-04-02
**Status:** Approved (v2 — expanded with quantifiable UX metrics)

## Overview

Expand the qa-skills plugin from 3 QA personas (smoke, ux, adversarial) to 5 by adding a **performance profiler agent** and a **mobile UX auditor agent**. Expand the existing **UX-auditor** with 2 new rubric categories and enhanced thresholds across 4 existing categories. Restructure all agents to use dedicated reference files. Update `/run-qa` to dispatch any combination of the 5 personas.

All UX and mobile checks produce **numeric values** enabling before/after measurement. Output includes a **binary scorecard** (Pass/Fail per check, total X/N) for quantified improvement tracking.

## Design Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Performance: skill vs agent | Agent replaces skill | Single artifact, consistent with other personas |
| Performance: fix code? | Report-only (measure + flag) | Keeps agent fast and non-destructive |
| Mobile: scope | Comprehensive (iOS native feel + mobile web best practices) | ~80 checks, covers both app types |
| Mobile: viewport | Single — 393x852 (iPhone 15 Pro) | Consistent with existing mobile workflow convention |
| `/run-qa` interface | Flat list, one at a time or any combination | `[smoke\|ux\|adversarial\|performance\|mobile\|all]` |
| Reference structure | One reference file per persona | Clean separation; agents stay focused on process/output |
| Agent architecture | Agent + shared references (Approach 2) | Matches existing plugin patterns |
| UX scoring | Binary scorecard (Pass=1/Fail=0) + severity grades | Enables quantified before/after comparison |

---

## Agent 1: Performance Profiler

**File:** `agents/performance-profiler.md`
**Reference:** `references/performance-profiler.md`
**Script:** `scripts/performance/parse-build-output.sh` (existing)

### Role

Measure runtime performance and static code patterns on each assigned screen/route. Report-only — no fixes, no PRs.

### Runtime Metrics (collected via Playwright `browser_evaluate`)

| Category | Metrics | Source |
|----------|---------|--------|
| Core Web Vitals | LCP, CLS, INP | PerformanceObserver |
| Supplemental Vitals | TTFB, FCP, TBT (via Long Tasks sum) | Navigation Timing + PerformanceObserver |
| DOM Health | Node count, max depth, max children per node | `document.querySelectorAll('*')` |
| Long Tasks | Count and duration of tasks > 50ms | PerformanceObserver `longtask` |
| Memory | JS heap size snapshot | `performance.memory` (Chromium) |
| Resource Loading | Total JS/CSS/image KB, request count, heavy resources > 50KB | Resource Timing API |

### Static Analysis

The existing 73 checks across 7 categories from the current performance-profiler skill:

1. Bundle & Code Splitting (10 checks)
2. Rendering & Hydration (8 checks)
3. API Routes & Data Fetching (10 checks)
4. Images & Assets (8 checks)
5. Third-party SDKs — PostHog (6), Sentry (6), Supabase (10)
6. Caching & Revalidation (7 checks)
7. Server Components & Streaming (8 checks)

### Flagging Thresholds

| Metric | MEDIUM | HIGH |
|--------|--------|------|
| LCP | — | > 2500ms |
| CLS | — | > 0.1 |
| INP | — | > 200ms |
| FCP | > 1800ms | > 3000ms |
| TTFB | > 800ms | > 1800ms |
| TBT | > 200ms | > 300ms |
| Total JS (compressed) | > 300KB | > 500KB |
| Total page weight | > 2000KB | > 4000KB |
| DOM nodes | > 1500 | > 3000 |
| Long Tasks count | > 3 | any single > 200ms |
| HTTP requests | > 50 | — |
| JS Execution Time | > 2s | > 3.5s |
| Lighthouse Performance score | < 90 | < 50 |

### Output Format

Per-route metrics table + findings list with severity ratings + binary scorecard.

```markdown
## Performance Profiler Results

### Scorecard: 9/13 Pass

### Per-Route Metrics
| Route | TTFB | FCP | LCP | CLS | INP | TBT | JS (KB) | DOM Nodes | Rating |
|-------|------|-----|-----|-----|-----|-----|---------|-----------|--------|
| / | 120ms | 1.2s | 1.8s | 0.02 | 85ms | 120ms | 245 | 890 | Good |
| /dashboard | 340ms | 2.1s | 3.2s | 0.18 | 220ms | 340ms | 412 | 2100 | Poor |

### Findings
1. [HIGH] **LCP 3.2s on /dashboard** — Largest element is an unoptimized hero image...
2. [HIGH] **CLS 0.18 on /dashboard** — Layout shift caused by...
3. [MEDIUM] **TBT 340ms on /dashboard** — 7 long tasks detected...
```

---

## UX-Auditor Expansion

The existing UX-auditor (8 categories, ~50 checks) gets **2 new categories** and **enhanced thresholds in 4 existing categories**, bringing total to **10 categories, ~75 checks**.

### New Category 9: Data Display & Scalability (~10 checks)

Addresses unpaginated lists, walls of content, missing search/filter patterns.

| Check | Good | Warning | Bad | Method |
|-------|------|---------|-----|--------|
| Page scroll depth ratio | <= 3x viewport | 3-5x | > 5x | `scrollHeight / clientHeight` |
| Repeated item count without pagination | <= 25 | 25-50 | > 50 | Count matching DOM structures by shared class/tag pattern |
| Pagination controls present | Present when items > 25 | Sort only | Absent | Query `[aria-label*="page"]`, `.pagination`, `nav` with page links |
| Search input present | Present when items > 50 | — | Absent | Query `input[type="search"]`, `[role="search"]` |
| Filter controls present | Present when items > 25 | — | Absent | Query select/dropdown/filter elements near list |
| Sticky header on long pages | Present when scroll > 3x viewport | — | Absent | Check `position: sticky/fixed` on headers when `scrollHeight > 3 * clientHeight` |
| Empty state quality | Explanation + CTA + visual | Text only | Blank screen | Check empty containers for heading, button, and img/svg children |
| Virtual scroll for large lists | Present when items > 200 | — | Absent | Check for IntersectionObserver / windowed rendering patterns |
| Scroll-to-action distance | Primary CTA within 2 viewports | — | > 2 viewports from top | Measure Y-position of primary CTA / `clientHeight` |
| Items-per-page count | 10-50 | 50-100 | > 100 or unbounded | Count visible repeated items |

### New Category 10: Visual Complexity & Consistency (~12 checks)

Measures design system adherence, visual clutter, and cross-screen consistency.

| Check | Good | Warning | Bad | Method |
|-------|------|---------|-----|--------|
| Distinct font sizes | <= 6 | 7-9 | > 10 | `getComputedStyle` `fontSize` across all text elements, count unique |
| Distinct font families | <= 2 | 3 | > 3 | `getComputedStyle` `fontFamily`, count unique |
| Distinct font-size/weight combos | <= 10 | 11-15 | > 15 | Composite key of `fontSize\|fontWeight\|fontFamily` |
| Distinct colors in use | <= 15 | 16-25 | > 25 | Unique `color` + `backgroundColor` values |
| Spacing grid conformance (4px) | > 90% | 70-90% | < 70% | Check all margin/padding values `% 4 === 0` |
| Alignment consistency | <= 5 left-edge clusters | 6-8 | > 8 | Cluster `getBoundingClientRect().left` values within 2px tolerance |
| Visual balance (Ngo score) | > 0.85 | 0.6-0.85 | < 0.6 | Area-weighted distance from center formula |
| Content line length | 45-75 chars | 75-90 or 30-45 | > 90 or < 30 | `containerWidth / (fontSize * 0.5)` |
| Whitespace ratio | 30-50% | 20-30% or > 60% | < 20% | Occupied element area / viewport area |
| Icon consistency (stroke/fill) | Uniform | — | Mixed outline + filled | SVG `stroke-width` and `fill` attribute audit |
| Icon sizing consistency | All same viewBox | — | Multiple sizes | SVG `viewBox` and client dimensions audit |
| Button style variations | 1-3 types | 4-5 | > 5 | Count distinct button visual patterns (bg color + border + radius combos) |

### Enhanced Existing Category: Navigation & Wayfinding (category 6)

Add these checks to the existing ones:

| Check | Good | Warning | Bad | Method |
|-------|------|---------|-----|--------|
| Primary nav item count | 5-7 | 8-9 | > 9 | Count `nav > ul > li` or `nav > a` direct children |
| Dropdown item count per group | <= 7 | 8-15 | > 15 | Count `option` / `[role="menuitem"]` children |
| Click depth to key pages | <= 3 | 4 | > 5 | Crawl navigation links, measure depth |
| Breadcrumbs at depth >= 3 | Present | — | Absent | Check URL segment count + `nav[aria-label*="breadcrumb"]` |
| CTA count per view | 1 primary | 2-3 | > 3 competing primary-styled buttons | Count prominent buttons in viewport |
| Back button fidelity | 100% of SPA transitions | — | < 80% | Test `history.back()` behavior |

### Enhanced Existing Category: Forms & Input (category 7)

Add these checks:

| Check | Good | Warning | Bad | Method |
|-------|------|---------|-----|--------|
| Visible fields per section | 3-5 | 6-7 | > 7 without grouping | Count visible inputs per fieldset/section |
| Error message position | Inline, adjacent to field | Top of form | Console/alert only | Check `aria-describedby` + proximity measurement |
| Error message actionability | Names field + how to fix (3-part) | Generic but present | Missing or "Error" | Text content analysis |
| Validation timing | On-blur | On-submit only | Premature (on-keypress before first blur) | Event listener analysis |
| Multi-step progress indicator | Present for > 5 fields | — | Absent | Check for step/progress UI when form has > 5 fields |
| Destructive action confirmation | Specific verb ("Delete project") | Generic "OK/Cancel" | No confirmation | Check dialog button text for specifics vs generic |
| Undo availability | Undo toast for reversible actions | Confirmation dialog | Neither | Check for undo UI patterns after destructive actions |

### Enhanced Existing Category: Feedback & Response (category 8)

Add these checks:

| Check | Good | Warning | Bad | Method |
|-------|------|---------|-----|--------|
| Skeleton screen presence | Skeleton for loads > 300ms | Spinner only | Blank screen | Query `[class*="skeleton"]`, `[class*="shimmer"]` |
| Blank screen time | 0ms | — | Any blank period | Check for content or loading indicator immediately after navigate |
| CLS during loading | 0 | < 0.1 | > 0.1 | PerformanceObserver `layout-shift` |
| Animation duration | 200-300ms | 100-500ms | > 500ms or < 50ms | CSS `transition-duration` / `animation-duration` audit |
| Toast/notification duration | 3-5s (5s+ with action button) | 2-3s | < 2s or no auto-dismiss for errors | Timing measurement on toast elements |
| Pull-to-refresh on scrollable lists | Present | — | Absent | Check for pull-to-refresh patterns on scrollable containers |
| Search-as-you-type latency | < 200ms | 200-500ms | > 500ms | Measure input event to DOM update timing |

### Enhanced Existing Category: Accessibility (category 4)

Add cognitive load checks:

| Check | Good | Warning | Bad | Method |
|-------|------|---------|-----|--------|
| Information density (words/viewport) | 150-300 | 300-500 | > 500 | `innerText` word count within viewport bounds |
| DOM element count | < 1500 | 1500-3000 | > 3000 | `querySelectorAll('*').length` |
| Choices per interaction context | <= 5 | 6-9 | > 9 visible actionable elements | Count buttons/links in focused section |
| Flesch-Kincaid grade level | 7-8th grade | 9-12th | > 12th | Compute from `innerText` using F-K formula |
| Flesch Reading Ease | 60-80 | 40-60 | < 40 | Compute from `innerText` using FRE formula |
| Heading frequency | Every 200-300 words | 300-600 words | > 600 words between headings | Count words between `h1`-`h6` elements |

### New UX-Auditor Output Format

Add binary scorecard to existing per-screen rubric:

```markdown
## UX Audit Results

### Scorecard: 62/75 Pass (83%)

### [Screen Name] — [URL]

| Category | Grade | Pass/Total | Findings |
|----------|-------|------------|----------|
| Visual Consistency | MINOR | 6/7 | 1 finding |
| Component States | PASS | 8/8 | — |
| Copy & Microcopy | PASS | 7/7 | — |
| Accessibility | MINOR | 8/10 | 2 findings |
| Layout & Responsiveness | PASS | 6/6 | — |
| Navigation & Wayfinding | MAJOR | 7/11 | 4 findings |
| Forms & Input | MINOR | 11/13 | 2 findings |
| Feedback & Response | PASS | 9/12 | — |
| Data Display & Scalability | CRITICAL | 3/10 | 7 findings |
| Visual Complexity & Consistency | MINOR | 10/12 | 2 findings |
```

---

## Agent 2: Mobile UX Auditor

**File:** `agents/mobile-ux-auditor.md`
**Reference:** `references/mobile-ux-auditor.md`
**Also references:** `references/ios-hig-requirements.md`, `references/ios-hig-anti-patterns.md`

### Role

Comprehensive mobile UX audit at 393x852 viewport. Covers iOS-native-feel standards and mobile web best practices. Report-only — inspects and grades, no fixes.

### Setup

At start of each screen: `browser_resize width=393 height=852`

### Category 1: Touch & Interaction (~7 checks)

| Check | Threshold | Method |
|-------|-----------|--------|
| Tap target size (Apple HIG) | >= 44x44 CSS px | `getBoundingClientRect()` on all interactive elements |
| Tap target size (Google MD3) | >= 48x48 CSS px (flag as info) | Same |
| Tap target spacing | >= 8px between adjacent targets | Compute gaps between interactive element rects |
| Icon-only nav items have text labels | aria-label or visible text present | Query nav items with SVG/img but no adjacent text |
| Primary CTAs in thumb zone | 100% of primary CTAs within bottom 60% of viewport | Measure Y-position of primary buttons; `y > 0.4 * viewportHeight` |
| Input field minimum height | >= 48px | `getBoundingClientRect().height` on all form inputs |
| Label visibility | Labels above fields, not placeholder-only | Check for `<label>` elements associated with inputs |

### Category 2: iOS Safari Specific (~5 checks)

| Check | What to detect | Method |
|-------|---------------|--------|
| `100vh` bug | Elements using `height: 100vh` | `browser_evaluate` computed styles scan |
| Input zoom prevention | `<input>`, `<select>`, `<textarea>` with font-size < 16px | Query + getComputedStyle |
| Safe area insets | If `viewport-fit=cover`, verify `env(safe-area-inset-*)` in CSS | Parse meta viewport + grep CSS |
| `position: fixed; bottom: 0` + keyboard | Fixed bottom elements that would overlap keyboard | Query fixed-positioned elements |
| Dynamic viewport units | Flag `100vh` usage, suggest `100dvh`/`svh` | CSS scan |

### Category 3: iOS Native Feel (~6 checks)

| Check | Anti-pattern | Best practice |
|-------|-------------|---------------|
| Hamburger menu detection | Hidden nav behind 3-line icon | Tab bar or visible nav (2-3x engagement improvement) |
| FAB detection | Floating circle button | Nav bar buttons |
| Breadcrumb on mobile | Desktop pattern on small screen | Back button + title |
| Material Design styling | Android-specific visuals (elevation, ripples) | iOS shadows, rounded corners |
| Component patterns | Web checkboxes, web dropdowns | iOS-style toggles, pickers |
| Toast/snackbar detection | Android-style bottom notifications | iOS alert/banner patterns |

### Category 4: Viewport & Responsive (~6 checks)

| Check | Threshold | Method |
|-------|-----------|--------|
| Viewport meta tag present | `width=device-width, initial-scale=1` | Parse `<meta name="viewport">` |
| Zoom not disabled | `user-scalable != no`, `maximum-scale >= 2` | Parse viewport meta content |
| No horizontal overflow | `scrollWidth <= clientWidth` | `browser_evaluate` |
| Orientation support | Content works in landscape (852x393) | Resize + check for content loss |
| Reflow at 320px | No horizontal scroll at 320px width (WCAG 1.4.10) | Resize to 320px + check |
| Viewport utilization | < 20% of viewport consumed by nav chrome | Measure fixed header/footer heights as % of `clientHeight` |

### Category 5: Mobile Typography (~10 checks)

| Check | Threshold | Method |
|-------|-----------|--------|
| Body text font size | >= 16px CSS (17pt iOS) | getComputedStyle on body text elements |
| Input font size | >= 16px (iOS zoom prevention) | getComputedStyle on form elements |
| Minimum any text | >= 11px (iOS caption2 floor) | getComputedStyle on all text elements |
| Line height | >= 1.5x font size (WCAG 1.4.12) | Computed line-height / font-size ratio |
| Line length (mobile) | 30-50 characters | `containerWidth / (fontSize * 0.5)` |
| Paragraph spacing | >= 2x font size | Computed margin-bottom |
| Letter spacing | >= 0.12x font size | Computed letter-spacing |
| Color contrast (normal text, AA) | >= 4.5:1 | Foreground vs background color extraction |
| Color contrast (large text, AA) | >= 3:1 (>= 18pt or >= 14pt bold) | Same, with size check |
| Text scaling at 200% | No truncation or clipping | Set zoom to 200%, check for overflow |

### Category 6: Mobile Form UX (~8 checks)

| Check | Requirement | Method |
|-------|-------------|--------|
| Email fields use `type="email"` | Fields with "email" in name/label have correct type | Query + attribute check |
| Phone fields use `type="tel"` | Fields with "phone"/"tel" in name/label | Same |
| Numeric fields use `inputmode` | `inputmode="numeric"` or `"decimal"` where appropriate | Same |
| `autocomplete` attributes present | 100% of applicable fields | Query all inputs, check attribute |
| `enterkeyhint` present | Form fields have enterkeyhint | Query inputs in forms |
| Single-column layout | No side-by-side form fields at mobile viewport | Detect form fields at same Y-position |
| Password fields | `autocomplete="current-password"` or `"new-password"` | Attribute check |
| Keyboard type matching | 100% of inputs show correct mobile keyboard | Cross-reference `type`/`inputmode` against field name/label content |

### Category 7: Interstitials & Overlays (~4 checks)

| Check | Threshold | Method |
|-------|-----------|--------|
| Overlay coverage on load | < 30% of viewport area | Detect fixed/absolute elements with high z-index, compute area ratio |
| Sticky banner height | < 15% of viewport | Measure fixed header/footer heights |
| Popup close button size | >= 44x44 CSS px | Find close buttons in overlay elements |
| Overlay timing | Flag overlays appearing within 3s of load | Observe DOM mutations after navigate |

### Category 8: Mobile Accessibility — WCAG Mobile (~6 checks)

| Check | WCAG ID | Method |
|-------|---------|--------|
| Touch targets >= 24x24 CSS px (AA) | 2.5.8 | getBoundingClientRect |
| Touch targets >= 44x44 CSS px (AAA) | 2.5.5 | Same (flag as enhancement) |
| `prefers-reduced-motion` support | — | Check for media query in stylesheets |
| Focus not obscured by sticky elements | 2.4.11 | Tab through elements, check visibility against fixed headers/footers |
| Hover-dependent UI has touch alternative | — | Check for `@media (hover: none)` or `(pointer: coarse)` rules |
| Text resize to 200% | 1.4.4 | Set zoom, check for overflow/clipping |

### Category 9: Gestures & Interaction (~5 checks)

| Check | Threshold | Method |
|-------|-----------|--------|
| Pull-to-refresh on scrollable lists | Present on all scrollable list screens | Check for pull-to-refresh patterns/indicators |
| Swipe-back gesture support | 100% of pushed screens | Navigate into sub-pages, test back-swipe behavior |
| Swipe-to-reveal on list items | Consistent across all applicable lists | Test swipe gesture on list items |
| Gesture cancellability | User can cancel mid-swipe | Test swipe + reverse direction |
| Skeleton screens for loads > 300ms | 100% of content-loading screens | Query `[class*="skeleton"]`, `[class*="shimmer"]` during navigation |

### Category 10: Animation & Motion (~5 checks)

| Check | Threshold | Method |
|-------|-----------|--------|
| Animation duration range | 100-600ms (MD3 standard range) | CSS `transition-duration` / `animation-duration` audit |
| No linear easing on spatial transforms | 0 violations | Check `transition-timing-function` on elements with `transform` transitions |
| Entrance vs exit asymmetry | Exit animations faster than entrance | Compare enter/exit duration values |
| Elevation consistency (MD3) | Standard levels: 0/1/3/6/8/12dp | Audit `box-shadow` values against MD3 spec |
| Tonal elevation preferred | Shadow-only elements flagged | Count elements using shadow without tonal surface color |

### Grading

Same as UX-auditor: PASS / MINOR / MAJOR / CRITICAL per category.

### Output Format (Binary Scorecard)

```markdown
## Mobile UX Audit Results

### Scorecard: 48/56 Pass (86%)

### [Screen Name] — [URL] (393x852)

| Category | Grade | Pass/Total | Findings |
|----------|-------|------------|----------|
| Touch & Interaction | MINOR | 6/7 | 1 finding |
| iOS Safari Specific | PASS | 5/5 | — |
| iOS Native Feel | MAJOR | 4/6 | 2 findings |
| Viewport & Responsive | PASS | 6/6 | — |
| Mobile Typography | MINOR | 8/10 | 2 findings |
| Mobile Form UX | MAJOR | 5/8 | 3 findings |
| Interstitials & Overlays | PASS | 4/4 | — |
| Mobile Accessibility | MINOR | 5/6 | 1 finding |
| Gestures & Interaction | PASS | 5/5 | — |
| Animation & Motion | MINOR | 4/5 | 1 finding |

### Findings Detail
1. [MAJOR] **Hamburger menu on /dashboard** — Primary navigation hidden behind...
2. [MAJOR] **Missing autocomplete on login form** — Email and password fields...
3. [MINOR] **Search input font-size 14px** — Below 16px threshold, will trigger iOS Safari zoom...
```

---

## `/run-qa` Command Update

**File:** `commands/run-qa.md`

### Argument Parsing

```
/run-qa [persona...] [--url URL]
```

Accepts any combination:

```
/run-qa smoke                    -> just smoke
/run-qa ux                       -> just ux
/run-qa adversarial              -> just adversarial
/run-qa performance              -> just performance
/run-qa mobile                   -> just mobile
/run-qa smoke ux mobile          -> any combination
/run-qa all                      -> all 5
```

### Dispatch

Each selected persona runs independently on assigned screens. No persona gates another.

### Report Format

Unified report with one section per dispatched persona:

```markdown
## QA Report — YYYY-MM-DD

### Smoke Test Results
(pass/fail per step)

### UX Audit Results
(scorecard X/75 + graded rubric per screen, 10 categories)

### Mobile UX Audit Results
(scorecard X/56 + graded rubric per screen, 10 mobile categories)

### Performance Results
(scorecard X/13 + per-route metrics table + flagged findings)

### Adversarial Results
(findings with severity)
```

Only sections for dispatched personas appear in the report.

---

## Reference File Structure

One reference file per persona:

| Persona | Reference File | Contents |
|---------|---------------|----------|
| Smoke | `references/smoke-tester.md` | Action mapping, pass/fail criteria, verification patterns |
| UX | `references/ux-auditor.md` | 10-category rubric with all check implementations, `browser_evaluate` measurement scripts, thresholds, Flesch-Kincaid formulas, Ngo balance formula |
| Adversarial | `references/adversarial-breaker.md` | Attack patterns, severity classification, reproduction templates |
| Performance | `references/performance-profiler.md` | Metrics collection scripts, 73 static checks, expanded thresholds |
| Mobile | `references/mobile-ux-auditor.md` | 10-category check implementations, `browser_evaluate` measurement scripts, MD3 motion/elevation specs, iOS type scale |

Mobile agent also references (read-only, no changes):
- `references/ios-hig-requirements.md`
- `references/ios-hig-anti-patterns.md`

## Scripts

| Persona | Script | Purpose |
|---------|--------|---------|
| Performance | `scripts/performance/parse-build-output.sh` (existing) | Parse `next build` output into structured metrics |

---

## File Change Summary

### New Files (7)

| File | Description |
|------|-------------|
| `agents/performance-profiler.md` | Performance agent — runtime metrics + static analysis, report-only |
| `agents/mobile-ux-auditor.md` | Mobile UX agent — 10 categories, ~56 checks, 393x852 viewport, binary scorecard |
| `references/performance-profiler.md` | Consolidated performance reference (metrics collection + 73 checks + thresholds) |
| `references/mobile-ux-auditor.md` | Mobile check implementations with `browser_evaluate` scripts, MD3 specs, iOS type scale |
| `references/smoke-tester.md` | Extracted from smoke agent inline content |
| `references/ux-auditor.md` | 10-category rubric with ~75 checks, measurement scripts, formulas |
| `references/adversarial-breaker.md` | Extracted from adversarial agent inline content |

### Updated Files (5)

| File | Changes |
|------|---------|
| `commands/run-qa.md` | Add performance + mobile to argument parsing, dispatch, report format with scorecards |
| `agents/smoke-tester.md` | Slim down — move detail to `references/smoke-tester.md` |
| `agents/ux-auditor.md` | Slim down — move detail to `references/ux-auditor.md`; add 2 new categories + enhanced checks |
| `agents/adversarial-breaker.md` | Slim down — move detail to `references/adversarial-breaker.md` |
| `.claude-plugin/plugin.json` | Update description to mention 5 personas |

### Deleted Files (5)

| File | Reason |
|------|--------|
| `skills/performance-profiler/SKILL.md` | Replaced by agent |
| `skills/performance-profiler/` directory | Entire skill directory removed |
| `references/performance-checks.md` | Consolidated into `references/performance-profiler.md` |
| `references/web-vitals-measurement.md` | Consolidated into `references/performance-profiler.md` |
| `scripts/performance/compare-metrics.sh` | No longer needed (report-only) |
