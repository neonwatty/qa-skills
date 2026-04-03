# Measurement Reliability: Structural + Script Fixes (Plan A)

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Make the qa-skills measurement stack more honest, reliable, and deterministic by tiering all checks, adding anti-gaming compound conditions, fixing 12 script reliability bugs, and updating the scorecard format to report confidence levels.

**Architecture:** Each reference file gets check-level tier tags (`[D]`/`[H]`/`[J]`), improved JavaScript measurement scripts, and a scoring section with graduated scoring, category weighting, and critical floor rules. Agent files get updated output format sections reflecting the new scorecard structure.

**Tech Stack:** Markdown reference files with embedded JavaScript `browser_evaluate` snippets. No new dependencies.

**Spec:** `docs/superpowers/specs/2026-04-02-measurement-reliability-improvements.md`

---

## File Map

| File | Changes |
|------|---------|
| `references/ux-auditor.md` | Add [D]/[H]/[J] tags to all ~99 checks, replace 12 measurement scripts with improved versions, add compound conditions, add graduated scoring section, add weighting + critical floor, add threshold citations, add conflicting checks documentation |
| `references/mobile-ux-auditor.md` | Add [D]/[H]/[J] tags to all ~56 checks, fix contrast ratio script, fix form keyword matching, fix hamburger SVG detection, add compound conditions, add graduated scoring, add weighting + critical floor, add threshold citations |
| `references/performance-profiler.md` | Add [D]/[H] tags to runtime metrics and static checks, fix CLS session window algorithm, fix Long Tasks observer, fix Navigation Timing guard, fix Resource Loading CSS detection, add page-settled detection, document Chromium-only limitations |
| `agents/ux-auditor.md` | Update output format to show tiered sub-scores |
| `agents/mobile-ux-auditor.md` | Update output format to show tiered sub-scores |
| `agents/performance-profiler.md` | Update output format to show tiered sub-scores, note Chromium-only metrics |

---

## Task 1: Add tier tags and scoring framework to UX-auditor reference

The largest single task. Tag every check in `references/ux-auditor.md` with `[D]`, `[H]`, or `[J]` and add the scoring framework sections.

**Files:**
- Modify: `references/ux-auditor.md`

- [ ] **Step 1: Read the full file**

Read `references/ux-auditor.md` in full.

- [ ] **Step 2: Add tier tags to every check**

For each checkbox line, prepend the tier tag. Use this classification:

**Category 1: Visual Consistency (7 checks)**
```
- [ ] `[H]` Typography: font sizes, weights, and line heights follow a consistent scale
- [ ] `[H]` Spacing: padding and margins use a consistent system (4px/8px grid or similar)
- [ ] `[H]` Colors: brand colors are used consistently, no off-by-one hex values
- [ ] `[H]` Border radii: consistent across similar elements (buttons, cards, inputs)
- [ ] `[H]` Shadows: consistent depth system, not arbitrary values
- [ ] `[J]` Icons: consistent style (outline vs filled), consistent sizing
- [ ] `[H]` Alignment: elements are properly aligned to a grid, no off-by-1px misalignment
```

**Category 2: Component States (8 checks)**
```
- [ ] `[J]` Default state: clear, not ambiguous
- [ ] `[H]` Hover state: present on all interactive elements, provides visual feedback
- [ ] `[D]` Focus state: visible focus ring for keyboard navigation (accessibility)
- [ ] `[J]` Active/pressed state: provides tactile feedback
- [ ] `[D]` Disabled state: visually distinct, not clickable
- [ ] `[H]` Loading state: present where async operations occur, uses consistent pattern
- [ ] `[J]` Empty state: helpful message and action when no data exists (not just blank space)
- [ ] `[J]` Error state: clear, specific, actionable error messages near the relevant field
```

**Category 3: Copy & Microcopy (7 checks)**
```
- [ ] `[H]` Error messages: specific ("Email is already registered") not vague ("Something went wrong")
- [ ] `[H]` Button labels: action-oriented ("Save Changes" not "Submit"), consistent capitalization
- [ ] `[H]` Placeholder text: helpful examples, not labels (labels should be above the field)
- [ ] `[H]` Confirmation messages: tell the user what happened ("Profile updated" not "Success")
- [ ] `[H]` Empty states: explain what goes here and how to add content
- [ ] `[J]` Tooltips: present where needed, concise, not redundant with visible labels
- [ ] `[H]` Grammar and spelling: no typos, consistent voice and tense
```

**Category 4: Accessibility (13 checks)**
```
- [ ] `[D]` Color contrast: text meets WCAG AA (4.5:1 for normal text, 3:1 for large)
- [ ] `[D]` Touch targets: at least 44x44px on interactive elements
- [ ] `[D]` Form labels: every input has an associated label (not just placeholder)
- [ ] `[D]` Alt text: images have meaningful alt text (or empty alt for decorative)
- [ ] `[D]` Heading hierarchy: h1 -> h2 -> h3, no skipped levels
- [ ] `[D]` Tab order: logical, follows visual flow
- [ ] `[H]` Screen reader: critical content is not conveyed by color alone
- [ ] `[D]` Information density (words/viewport): 150-300 good, 300-500 warning, >500 bad
- [ ] `[D]` DOM element count: <1500 good, 1500-3000 warning, >3000 bad
- [ ] `[H]` Choices per interaction context: <=5 good, 6-9 warning, >9 bad
- [ ] `[D]` Flesch-Kincaid grade level: 7-8th good, 9-12th warning, >12th bad
- [ ] `[D]` Flesch Reading Ease: 60-80 good, 40-60 warning, <40 bad
- [ ] `[D]` Heading frequency: every 200-300 words good, 300-600 warning, >600 bad
```

**Category 5: Layout & Responsiveness (6 checks)**
```
- [ ] `[D]` Content width: readable line length (45-75 characters for body text)
- [ ] `[D]` Viewport fit: no horizontal scroll at the current viewport
- [ ] `[D]` Element overflow: text truncates gracefully (ellipsis, not clip)
- [ ] `[H]` Image sizing: images are properly constrained, no layout shift on load
- [ ] `[J]` Whitespace: balanced, no cramped or excessively empty areas
- [ ] `[J]` Z-index: overlapping elements stack correctly (dropdowns, modals, tooltips)
```

**Category 6: Navigation & Wayfinding (11 checks)**
```
- [ ] `[D]` Current location: user knows where they are (breadcrumbs, active nav state, page title)
- [ ] `[H]` Back navigation: browser back button works as expected
- [ ] `[H]` URL reflects state: deep-linkable, shareable
- [ ] `[H]` Dead ends: no pages without a clear next action or way to navigate away
- [ ] `[D]` Breadcrumbs: present on nested pages, clickable
- [ ] `[D]` Primary nav item count: 5-7 good, 8-9 warning, >9 bad
- [ ] `[D]` Dropdown item count per group: <=7 good, 8-15 warning, >15 bad
- [ ] `[H]` Click depth to key pages: <=3 good, 4 warning, >5 bad
- [ ] `[D]` Breadcrumbs at depth >= 3: present = good, absent = bad
- [ ] `[D]` CTA count per view: 1 primary good, 2-3 warning, >3 competing bad
- [ ] `[H]` Back button fidelity: 100% good, <80% bad
```

**Category 7: Forms & Input (13 checks)**
```
- [ ] `[H]` Validation timing: inline validation on blur, not only on submit
- [ ] `[D]` Required indicators: clear marking of required fields
- [ ] `[D]` Input types: correct HTML input types (email, tel, number, url)
- [ ] `[D]` Autofill: standard fields work with browser autofill
- [ ] `[D]` Multi-step forms: progress indicator, ability to go back
- [ ] `[H]` Destructive actions: confirmation before irreversible operations
- [ ] `[D]` Visible fields per section: 3-5 good, 6-7 warning, >7 bad
- [ ] `[D]` Error message position: inline = good, top of form = warning, console/alert = bad
- [ ] `[J]` Error message actionability: names field + fix = good, generic = warning, missing = bad
- [ ] `[H]` Validation timing (granular): on-blur = good, on-submit only = warning, premature = bad
- [ ] `[D]` Multi-step progress indicator: present for >5 fields = good, absent = bad
- [ ] `[H]` Destructive action confirmation: specific verb = good, generic OK/Cancel = warning, none = bad
- [ ] `[J]` Undo availability: undo toast = good, confirmation only = warning, neither = bad
```

**Category 8: Feedback & Response (12 checks)**
```
- [ ] `[J]` Action feedback: every user action gets visible confirmation
- [ ] `[H]` Loading indicators: present during async operations, appropriate type
- [ ] `[J]` Optimistic updates: UI responds immediately where appropriate
- [ ] `[H]` Error recovery: clear path to retry or correct after errors
- [ ] `[H]` Success confirmation: user knows the action completed
- [ ] `[D]` Skeleton screen presence: skeleton for loads >300ms = good, spinner only = warning, blank = bad
- [ ] `[H]` Blank screen time: 0ms = good, any blank period = bad
- [ ] `[D]` CLS during loading: 0 = good, <0.1 = warning, >0.1 = bad
- [ ] `[D]` Animation duration: 200-300ms = good, 100-500ms = warning, >500ms = bad
- [ ] `[H]` Toast/notification duration: 3-5s = good, <2s or no auto-dismiss for errors = bad
- [ ] `[H]` Pull-to-refresh on scrollable lists: present = good, absent = bad
- [ ] `[H]` Search-as-you-type latency: <200ms good, 200-500ms warning, >500ms bad
```

**Category 9: Data Display & Scalability (10 checks)**
```
- [ ] `[D]` Page scroll depth ratio: <=3x good, 3-5x warning, >5x bad
- [ ] `[D]` Repeated item count without pagination: <=25 good, 25-50 warning, >50 bad
- [ ] `[D]` Pagination controls present: present when items >25 = good, absent = bad
- [ ] `[D]` Search input present: present when items >50 = good, absent = bad
- [ ] `[D]` Filter controls present: present when items >25 = good, absent = bad
- [ ] `[D]` Sticky header on long pages: present when scroll >3x = good, absent = bad
- [ ] `[J]` Empty state quality: explanation + CTA + visual = good, text only = warning, blank = bad
- [ ] `[H]` Virtual scroll for large lists: present when items >200 = good, absent = bad
- [ ] `[D]` Scroll-to-action distance: CTA within 2 viewports = good, >2 = bad
- [ ] `[D]` Items-per-page count: 10-50 good, 50-100 warning, >100 bad
```

**Category 10: Visual Complexity & Consistency (12 checks)**
```
- [ ] `[D]` Distinct font sizes: <=6 good, 7-9 warning, >10 bad
- [ ] `[D]` Distinct font families: <=2 good, 3 warning, >3 bad
- [ ] `[D]` Distinct font-size/weight combos: <=10 good, 11-15 warning, >15 bad
- [ ] `[D]` Distinct colors in use: <=15 good, 16-25 warning, >25 bad
- [ ] `[D]` Spacing grid conformance (4px): >90% good, 70-90% warning, <70% bad
- [ ] `[H]` Alignment consistency: <=5 left-edge clusters good, 6-8 warning, >8 bad
- [ ] `[H]` Visual balance (Ngo score): >0.85 good, 0.6-0.85 warning, <0.6 bad
- [ ] `[D]` Content line length: 45-75 chars good, 75-90 or 30-45 warning, >90 or <30 bad
- [ ] `[H]` Whitespace ratio: 30-50% good, 20-30% warning, <20% bad
- [ ] `[J]` Icon consistency (stroke/fill): uniform = good, mixed = bad
- [ ] `[D]` Icon sizing consistency: all same viewBox = good, multiple = bad
- [ ] `[H]` Button style variations: 1-3 good, 4-5 warning, >5 bad
```

- [ ] **Step 3: Add a tier legend at the top of the file**

After the introduction paragraph, add:

```markdown
## Measurement Tier Legend

Each check is tagged with its measurement confidence level:

- **`[D]` Deterministic** — Fully measurable via `browser_evaluate`. Returns a numeric value with a clear threshold. Same page always produces the same result. High confidence.
- **`[H]` Heuristic** — Measurable via `browser_evaluate` but with known false positive/negative risks (<5% error rate), OR requires Playwright interaction sequence (multi-step). Reliable signal, not definitive.
- **`[J]` LLM-Judgment** — Requires visual interpretation or semantic understanding. The agent examines screenshots or reads content. A programmatic pre-filter narrows what the LLM evaluates by 75-85%. Lower confidence.

### Threshold Citation Legend

Each threshold is tagged with its source:

- **`[research]`** — Backed by peer-reviewed research or official standards (WCAG, Apple HIG, MD3, Google Web Vitals)
- **`[convention]`** — Widely accepted industry convention (design system best practices, NNG guidance)
- **`[heuristic]`** — Team-chosen threshold based on experience. Reasonable but not externally validated.
```

- [ ] **Step 4: Add threshold citations to each check**

For each check that has a threshold, append the citation tag. Examples:
```
- [ ] `[D]` Color contrast: text meets WCAG AA (4.5:1 for normal text, 3:1 for large) `[research: WCAG 2.1 SC 1.4.3]`
- [ ] `[D]` Distinct font sizes: <=6 good, 7-9 warning, >10 bad `[convention: typography best practice]`
- [ ] `[D]` Page scroll depth ratio: <=3x good, 3-5x warning, >5x bad `[heuristic]`
```

Use the classification from the spec's "Threshold Citation Status" section.

- [ ] **Step 5: Add Scoring Framework section**

After the measurement scripts section, add a new section:

```markdown
## Scoring Framework

### Tiered Sub-Scores

Report separate scores for each measurement tier:

| Tier | Description | Confidence |
|------|-------------|-----------|
| Deterministic | Programmatic, reproducible | High |
| Heuristic | Programmatic with <5% error | Medium |
| LLM-Assisted | Pre-filtered LLM judgment | Lower |

### Graduated Scoring

These checks use 0 / 0.5 / 1.0 instead of binary pass/fail:

| Check | 0 | 0.5 | 1.0 |
|-------|---|-----|-----|
| Touch target size | < 24px | 24-43px | >= 44px |
| Color contrast | < 2:1 | 2:1-4.4:1 | >= 4.5:1 |
| Scroll depth ratio | > 8x | 3-8x | <= 3x |
| Font sizes count | > 12 | 7-12 | <= 6 |
| Information density | > 700 words | 300-700 | 150-300 |
| Animation duration | > 1500ms | 600-1500ms | 100-600ms |
| Line length | > 100 or < 20 chars | 75-100 or 20-30 | 30-75 |

### Category Weighting

| Weight | Categories |
|--------|-----------|
| 2x | Accessibility (Cat 4) |
| 1.5x | Forms & Input (Cat 7) |
| 1x | All other categories |
| 0.5x | Visual Complexity & Consistency (Cat 10) |

### Critical Floor Rule

If ANY check tagged as CRITICAL-severity fails, the total weighted score is capped at 50% regardless of other passes.

### Compound Conditions

These checks require multiple conditions to pass (prevents gaming):

1. **Autofill (Cat 7)**: `autocomplete` attribute present AND value is a valid HTML autofill token (not "on", "off", or empty)
2. **Touch target (Cat 4)**: `getBoundingClientRect()` >= 44x44 AND visible content area >= 30x30
3. **Pagination (Cat 9)**: Pagination DOM elements present AND has > 1 navigable page link
4. **Form label (Cat 4)**: `<label>` associated with input AND label is visible (rect area > 0) AND positioned above or beside field
5. **Hover state (Cat 2)**: Computed style changes on :hover AND at least one perceivable property change (color, backgroundColor, borderColor, boxShadow, transform, opacity delta > 0.1)
6. **Skeleton screen (Cat 8)**: Skeleton class/element found AND element has visible dimensions (width > 0, height > 0) AND has CSS animation
7. **Heading frequency (Cat 4)**: Headings every 200-300 words AND heading texts are not all identical

### Conflicting Checks

These check pairs can conflict. When both fail, report them as a linked pair rather than independent findings:

1. Whitespace ratio (Cat 10) vs scroll depth (Cat 9)
2. Touch target size (Cat 4) vs nav item count (Cat 6) on mobile
3. CTA count = 1 (Cat 6) vs empty state CTA (Cat 9)
4. Information density (Cat 4) vs heading frequency (Cat 4)
5. Font sizes <= 6 (Cat 10) vs responsive type scales

### Output Template (Updated)

```
## UX Audit Results

### Scorecard: X/Y Weighted (Z%)

| Tier | Pass/Total | Confidence |
|------|------------|-----------|
| Deterministic | 30/33 | High |
| Heuristic | 18/22 | Medium |
| LLM-Assisted | 8/10 | Lower |
| **Weighted Total** | **X/Y** | |

### [Screen Name] — [URL]

| Category | Weight | Grade | Pass/Total | Findings |
|----------|--------|-------|------------|----------|
| Visual Consistency | 1x | MINOR | 6/7 | 1 finding |
| Component States | 1x | PASS | 8/8 | -- |
| ...
```
```

- [ ] **Step 6: Commit**

```bash
git add references/ux-auditor.md
git commit -m "feat: add measurement tiers, scoring framework, and threshold citations to UX-auditor reference"
```

---

## Task 2: Fix UX-auditor measurement scripts

Replace the 12 scripts in `references/ux-auditor.md` that have known reliability issues.

**Files:**
- Modify: `references/ux-auditor.md` (Measurement Scripts section)

- [ ] **Step 1: Replace Spacing Grid Conformance script**

Find the existing `Spacing Grid Conformance` script and replace with the sub-pixel-tolerant version. Key change: `Math.round(val) % 4 === 0` instead of `val % 4 === 0`. Also skip `display: none` elements and handle negative margins.

- [ ] **Step 2: Replace Alignment Clustering script**

Replace with block-level-only version. Key changes:
- Filter to block-level display values only (block, flex, grid, table, list-item)
- Use explicit block element selectors
- Use cluster-start tracking instead of adjacent-value comparison

- [ ] **Step 3: Replace Visual Balance (Ngo) script**

Replace with leaf-only + viewport-clipped version. Key changes:
- Process elements deepest-first, skip parents whose children were already counted
- Clip all elements to viewport bounds
- Only count elements with `rect.bottom > 0 && rect.top < vh`

- [ ] **Step 4: Replace Whitespace Ratio script**

Replace with pixel-sampling approach. Key change: use `document.elementFromPoint()` on a 20px grid instead of summing bounding rect areas. This eliminates the double-counting problem entirely.

- [ ] **Step 5: Replace Font Audit script**

Replace with leaf-text-only version. Key changes:
- Use TreeWalker with `SHOW_TEXT` filter to find leaf text nodes
- Get computed style from `parentElement` of each text node
- Skip elements with `rect.width === 0 || rect.height === 0`
- Round font sizes to 0.1px to handle sub-pixel rendering

- [ ] **Step 6: Replace Repeated Item Count script**

Replace with version that detects div-grid patterns. Key addition: find any container whose children share a common class pattern (>= 50% of children), not just `ul/ol/table`.

- [ ] **Step 7: Replace Heading Frequency script**

Replace with version that filters `<script>` and `<style>` text nodes via NodeFilter callback. Also add duplicate heading detection (for compound condition).

- [ ] **Step 8: Replace Information Density script**

Replace with TreeWalker-based version that counts words from text nodes only (not element `innerText`), with viewport bounds checking on parent elements. Eliminates double-counting.

- [ ] **Step 9: Replace Nav/CTA Count script**

Replace CTA detection with improved version that excludes nav/footer buttons and classifies primary CTAs by explicit class or styling (not just "has a non-transparent background").

- [ ] **Step 10: Replace Flesch-Kincaid script**

Replace with version that extracts text from `<main>` (or `<article>`, `[role="main"]`) only, excluding `<nav>`, `<footer>`, `<header>`, `<script>`, `<style>`. Add `lang` attribute check — skip readability scoring for non-English pages.

- [ ] **Step 11: Replace Color Audit script**

Replace with version that normalizes colors by parsing to `{r, g, b}` tuples and grouping within Delta-E < 3.0 (just-noticeable difference). Report both raw distinct count and clustered count.

- [ ] **Step 12: Commit**

```bash
git add references/ux-auditor.md
git commit -m "fix: replace 11 UX-auditor measurement scripts with improved versions"
```

---

## Task 3: Add tier tags and scoring framework to mobile UX-auditor reference

Same pattern as Task 1 but for `references/mobile-ux-auditor.md`.

**Files:**
- Modify: `references/mobile-ux-auditor.md`

- [ ] **Step 1: Read the full file**

Read `references/mobile-ux-auditor.md` in full.

- [ ] **Step 2: Add tier tags to all 56 checks**

Use this classification:

**Category 1: Touch & Interaction (7 checks)**: 5 `[D]`, 2 `[D]`
- `[D]` Tap target size (Apple HIG), `[D]` Tap target size (Google MD3), `[D]` Tap target spacing, `[D]` Icon-only labels, `[D]` Primary CTAs in thumb zone, `[D]` Input field height, `[D]` Label visibility

**Category 2: iOS Safari Specific (5 checks)**: 3 `[D]`, 2 `[H]`
- `[D]` 100vh bug, `[D]` Input zoom prevention, `[H]` Safe area insets, `[D]` Fixed bottom + keyboard, `[H]` Dynamic viewport units

**Category 3: iOS Native Feel (6 checks)**: 1 `[H]`, 5 `[H]`
- `[H]` Hamburger detection, `[H]` FAB detection, `[H]` Breadcrumb on mobile, `[J]` Material Design styling, `[J]` Component patterns (checkbox vs toggle), `[H]` Toast/snackbar detection

**Category 4: Viewport & Responsive (6 checks)**: 5 `[D]`, 1 `[D]`
- `[D]` Viewport meta tag, `[D]` Zoom not disabled, `[D]` No horizontal overflow, `[H]` Orientation support, `[D]` Reflow at 320px, `[D]` Viewport utilization

**Category 5: Mobile Typography (10 checks)**: 8 `[D]`, 1 `[D]`, 1 `[H]`
- `[D]` Body text font size, `[D]` Input font size, `[D]` Minimum any text, `[D]` Line height, `[D]` Line length, `[D]` Paragraph spacing, `[D]` Letter spacing, `[D]` Color contrast normal, `[D]` Color contrast large, `[H]` Text scaling at 200%

**Category 6: Mobile Form UX (8 checks)**: 7 `[D]`, 1 `[D]`
- All 8 checks are `[D]` (DOM attribute checks)

**Category 7: Interstitials & Overlays (4 checks)**: 3 `[D]`, 1 `[H]`
- `[D]` Overlay coverage, `[D]` Sticky banner height, `[D]` Close button size, `[H]` Overlay timing

**Category 8: Mobile Accessibility (6 checks)**: 3 `[D]`, 2 `[H]`, 1 `[H]`
- `[D]` Touch targets AA, `[D]` Touch targets AAA, `[H]` prefers-reduced-motion, `[H]` Focus not obscured, `[H]` Hover-dependent UI alternative, `[H]` Text resize to 200%

**Category 9: Gestures & Interaction (5 checks)**: 1 `[D]`, 1 `[H]`, 3 `[J]`
- `[D]` Skeleton screens for loads, `[H]` Pull-to-refresh, `[J]` Swipe-back gesture, `[J]` Swipe-to-reveal, `[J]` Gesture cancellability

**Category 10: Animation & Motion (5 checks)**: 2 `[D]`, 2 `[H]`, 1 `[J]`
- `[D]` Animation duration range, `[D]` No linear easing on spatial, `[J]` Entrance/exit asymmetry, `[H]` Elevation consistency, `[H]` Tonal elevation preferred

- [ ] **Step 3: Add tier legend, threshold citations, and scoring framework**

Same structure as Task 1 Step 3-5, adapted for mobile categories. Mobile weighting:

| Weight | Categories |
|--------|-----------|
| 2x | Touch & Interaction (Cat 1), Mobile Accessibility (Cat 8) |
| 1.5x | Mobile Typography (Cat 5), iOS Safari Specific (Cat 2), Interstitials (Cat 7), Mobile Form UX (Cat 6) |
| 1x | Viewport & Responsive (Cat 4), iOS Native Feel (Cat 3) |
| 0.5x | Gestures & Interaction (Cat 9), Animation & Motion (Cat 10) |

Compound conditions for mobile:
1. Touch target >= 44x44 AND visible content >= 30x30
2. `prefers-reduced-motion` present AND modifies animation/transition
3. `autocomplete` valid token
4. Skeleton screen visible AND has animation

- [ ] **Step 4: Commit**

```bash
git add references/mobile-ux-auditor.md
git commit -m "feat: add measurement tiers, scoring framework, and threshold citations to mobile UX-auditor reference"
```

---

## Task 4: Fix mobile UX-auditor measurement scripts

Fix the 4 scripts with known reliability issues in `references/mobile-ux-auditor.md`.

**Files:**
- Modify: `references/mobile-ux-auditor.md`

- [ ] **Step 1: Fix contrast ratio script (Script 7)**

Key changes:
- Replace string-based transparency check (`bg.includes('rgba') && bg.includes(', 0)')`) with numeric alpha parsing
- Handle alpha channel blending: compute effective foreground color against background
- Detect `background-image` on ancestors and flag as `{ indeterminate: true, reason: 'background-image detected' }` instead of false-passing
- Add `button, input, textarea, select, figcaption, blockquote, dt, dd` to the text element selector
- Create color parser as a regex-based function (not temporary DOM div insertion)

- [ ] **Step 2: Fix form attribute validation script (Script 5)**

Key change: replace `combined.includes(keyword)` with `new RegExp('\\b' + keyword + '\\b', 'i').test(combined)` to prevent "tel" matching "hotel".

- [ ] **Step 3: Fix hamburger menu detection script (Script 8)**

Key change for SVG heuristic: require lines to be approximately horizontal (similar `y1`/`y2` values) and evenly spaced, not just "2-4 lines in an SVG". Add check that the button toggles a nav element (has `aria-controls` pointing to a `<nav>` or `[role="navigation"]`).

- [ ] **Step 4: Fix viewport meta tag parsing (Script 2)**

Key change: parse content string into key-value pairs and compare `initial-scale` numerically instead of using `content.includes('initial-scale=1')` which misses `initial-scale=1.0`.

- [ ] **Step 5: Fix overlay coverage script (Script 6)**

Key changes:
- Lower z-index threshold from 1000 to a heuristic: any `position: fixed` element covering > 10% viewport
- Combine the two `querySelectorAll('*')` passes into one
- Compute union of overlay rectangles instead of summing individual areas

- [ ] **Step 6: Commit**

```bash
git add references/mobile-ux-auditor.md
git commit -m "fix: improve contrast ratio, form validation, hamburger detection, viewport parsing, and overlay coverage scripts"
```

---

## Task 5: Fix performance profiler measurement scripts

Fix the 5 scripts with known reliability issues in `references/performance-profiler.md`.

**Files:**
- Modify: `references/performance-profiler.md`

- [ ] **Step 1: Read the full file**

Read `references/performance-profiler.md` in full.

- [ ] **Step 2: Add tier tags to all metrics and checks**

Tag each runtime metric and static check:
- All runtime metrics (TTFB, FCP, LCP, CLS, TBT, DOM, memory, resources): `[D]`
- Note Chromium-only metrics: LCP, CLS, Long Tasks/TBT, memory — add `[D, Chromium-only]`
- All 73 static checks: `[D]` (code pattern matching is deterministic)

- [ ] **Step 3: Fix CLS script to use session windows**

Replace the current CLS collection script (Step 4) with one implementing the web.dev session window algorithm:
- Group layout shifts within 1s gaps
- Maximum 5s from first shift in window
- Report the largest session window value, not the sum of all shifts

- [ ] **Step 4: Fix Long Tasks / TBT script**

Key changes:
- Add `try/catch` around `PerformanceObserver` creation
- Return `{ available: false, reason: 'longtask observer not supported' }` on non-Chromium
- Add note that `buffered: true` is not reliable for `longtask` — document that TBT only covers the observation window
- Increase observation window from 5s to 8s

- [ ] **Step 5: Fix Navigation Timing script**

Add guard: `if (nav.loadEventEnd === 0) return { available: true, partial: true, note: 'load event not yet complete', ... }` — return partial data instead of negative values.

- [ ] **Step 6: Fix Resource Loading CSS detection**

Replace `r.initiatorType === 'link' || r.initiatorType === 'css'` with a check that also filters by URL extension: `(r.initiatorType === 'link' && r.name.includes('.css')) || r.initiatorType === 'css'`. Also add a field counting resources with `transferSize === 0` (opaque cross-origin responses).

- [ ] **Step 7: Add LCP script fix**

Remove the `getEntriesByType('largest-contentful-paint')` fast path (unreliable across browsers). Always use PerformanceObserver with `buffered: true` and `try/catch`. Increase fallback timeout from 3s to 5s.

- [ ] **Step 8: Add page-settled detection recommendation**

Add a new section "Page Settled Detection" with recommended Playwright sequence to run before any measurement:

```markdown
## Page Settled Detection

Before running any measurement script, ensure the page is settled:

1. `browser_wait_for` expected content with 10s timeout
2. Wait 2 seconds for lazy loading and layout shifts
3. Optionally run this settle check:

```javascript
browser_evaluate:
(() => {
  return {
    readyState: document.readyState,
    fontsReady: document.fonts.status === 'loaded',
    runningAnimations: document.getAnimations().filter(a => a.playState === 'running').length,
    pendingImages: [...document.images].filter(img => !img.complete).length
  };
})()
```

If `runningAnimations > 0` or `pendingImages > 0`, wait an additional 2 seconds and re-check.
```

- [ ] **Step 9: Add Chromium-only documentation**

Add a section "Browser Compatibility" listing which metrics require Chromium:

```markdown
## Browser Compatibility

| Metric | Chromium | Firefox | WebKit |
|--------|----------|---------|--------|
| Navigation Timing | Yes | Yes | Yes |
| FCP (paint entries) | Yes | Yes | Yes |
| LCP | Yes | No | No |
| CLS (layout-shift) | Yes | No | No |
| Long Tasks / TBT | Yes | No | No |
| Memory | Yes | No | No |
| Resource Timing | Yes | Yes | Yes |
| DOM Health | Yes | Yes | Yes |

Scripts for Chromium-only metrics return `{ available: false, reason: '...' }` on unsupported browsers. The profiler should default to Chromium for comprehensive measurement.
```

- [ ] **Step 10: Commit**

```bash
git add references/performance-profiler.md
git commit -m "fix: CLS session windows, Long Tasks reliability, Navigation Timing guard, Resource Loading, LCP, page-settled detection, browser compatibility docs"
```

---

## Task 6: Update agent output formats

Update all three agent files to reflect the new tiered scorecard format.

**Files:**
- Modify: `agents/ux-auditor.md`
- Modify: `agents/mobile-ux-auditor.md`
- Modify: `agents/performance-profiler.md`

- [ ] **Step 1: Update UX-auditor output format**

In `agents/ux-auditor.md`, find the Output Format section and replace the scorecard template with:

```markdown
## UX Audit Results

### Scorecard: X/Y Weighted (Z%)

| Tier | Pass/Total | Confidence |
|------|------------|-----------|
| Deterministic [D] | 30/33 | High |
| Heuristic [H] | 18/22 | Medium |
| LLM-Assisted [J] | 8/10 | Lower |
| **Weighted Total** | **X/Y** | |

### [Screen Name] — [URL]

| Category | Weight | Grade | Pass/Total | Findings |
|----------|--------|-------|------------|----------|
| Visual Consistency | 1x | MINOR | 6/7 | 1 finding |
| ...
```

Also add a note: "Read `references/ux-auditor.md` for the complete scoring framework including graduated scoring, category weighting, critical floor rule, and compound conditions."

- [ ] **Step 2: Update mobile UX-auditor output format**

Same pattern for `agents/mobile-ux-auditor.md`. Replace scorecard with tiered version.

- [ ] **Step 3: Update performance profiler output format**

In `agents/performance-profiler.md`, add a note about Chromium-only metrics:

```markdown
### Scorecard: X/13 Pass

**Note:** Metrics marked [Chromium-only] return `available: false` on Firefox/WebKit. The scorecard denominator adjusts to only count available metrics.
```

- [ ] **Step 4: Commit**

```bash
git add agents/ux-auditor.md agents/mobile-ux-auditor.md agents/performance-profiler.md
git commit -m "feat: update agent output formats with tiered scorecards and confidence levels"
```

---

## Task 7: Run validation

**Files:**
- Run: `scripts/validate-skills.sh`
- Run: `npm run validate`

- [ ] **Step 1: Run skill/agent/command validation**

```bash
./scripts/validate-skills.sh
```

Expected: all 6 agents, 12 skills, 2 commands pass.

- [ ] **Step 2: Run full validation suite**

```bash
npm run validate
```

Expected: markdown lint + skill validation + link check all pass.

- [ ] **Step 3: Fix any issues and commit**

```bash
git add -A
git commit -m "fix: address validation issues from measurement reliability changes"
```

---

## Task Summary

| Task | Description | Dependencies | Estimated Size |
|------|-------------|--------------|----------------|
| 1 | Tier tags + scoring framework for UX-auditor | None | Large (622-line file) |
| 2 | Fix 11 UX-auditor measurement scripts | Task 1 | Large (replace 11 JS snippets) |
| 3 | Tier tags + scoring framework for mobile UX-auditor | None | Large (971-line file) |
| 4 | Fix 5 mobile measurement scripts | Task 3 | Medium |
| 5 | Fix performance profiler scripts + docs | None | Medium |
| 6 | Update 3 agent output formats | Tasks 1, 3, 5 | Small |
| 7 | Validation | All | Small |

**Parallelizable groups:**
- Group A: Tasks 1 + 2 (UX-auditor)
- Group B: Tasks 3 + 4 (mobile)
- Group C: Task 5 (performance)
- Group D: Task 6 (depends on A, B, C)
- Group E: Task 7 (depends on all)
