# UX Auditor — Complete Rubric Reference

This reference contains all 10 audit categories, measurement scripts, thresholds, and grading criteria for the UX auditor agent. It covers ~75 checks across Visual Consistency, Component States, Copy & Microcopy, Accessibility, Layout & Responsiveness, Navigation & Wayfinding, Forms & Input, Feedback & Response, Data Display & Scalability, and Visual Complexity & Consistency.

Each check maps to a binary Pass (1) or Fail (0). The total score per screen is X/75. Severity grades (PASS / MINOR / MAJOR / CRITICAL) are assigned per category based on the number and impact of failures.

---

## Category 1: Visual Consistency

- [ ] Typography: font sizes, weights, and line heights follow a consistent scale
- [ ] Spacing: padding and margins use a consistent system (4px/8px grid or similar)
- [ ] Colors: brand colors are used consistently, no off-by-one hex values
- [ ] Border radii: consistent across similar elements (buttons, cards, inputs)
- [ ] Shadows: consistent depth system, not arbitrary values
- [ ] Icons: consistent style (outline vs filled), consistent sizing
- [ ] Alignment: elements are properly aligned to a grid, no off-by-1px misalignment

---

## Category 2: Component States

- [ ] Default state: clear, not ambiguous
- [ ] Hover state: present on all interactive elements, provides visual feedback
- [ ] Focus state: visible focus ring for keyboard navigation (accessibility)
- [ ] Active/pressed state: provides tactile feedback
- [ ] Disabled state: visually distinct, not clickable
- [ ] Loading state: present where async operations occur, uses consistent pattern
- [ ] Empty state: helpful message and action when no data exists (not just blank space)
- [ ] Error state: clear, specific, actionable error messages near the relevant field

---

## Category 3: Copy & Microcopy

- [ ] Error messages: specific ("Email is already registered") not vague ("Something went wrong")
- [ ] Button labels: action-oriented ("Save Changes" not "Submit"), consistent capitalization
- [ ] Placeholder text: helpful examples, not labels (labels should be above the field)
- [ ] Confirmation messages: tell the user what happened ("Profile updated" not "Success")
- [ ] Empty states: explain what goes here and how to add content
- [ ] Tooltips: present where needed, concise, not redundant with visible labels
- [ ] Grammar and spelling: no typos, consistent voice and tense

---

## Category 4: Accessibility

### Existing Checks

- [ ] Color contrast: text meets WCAG AA (4.5:1 for normal text, 3:1 for large)
- [ ] Touch targets: at least 44x44px on interactive elements
- [ ] Form labels: every input has an associated label (not just placeholder)
- [ ] Alt text: images have meaningful alt text (or empty alt for decorative)
- [ ] Heading hierarchy: h1 -> h2 -> h3, no skipped levels
- [ ] Tab order: logical, follows visual flow
- [ ] Screen reader: critical content is not conveyed by color alone

### Cognitive Load Checks (Enhanced)

- [ ] Information density (words/viewport): 150-300 good, 300-500 warning, >500 bad. Method: `innerText` word count within viewport bounds
- [ ] DOM element count: <1500 good, 1500-3000 warning, >3000 bad. Method: `querySelectorAll('*').length`
- [ ] Choices per interaction context: <=5 good, 6-9 warning, >9 bad. Method: count visible actionable elements (buttons/links) in focused section
- [ ] Flesch-Kincaid grade level: 7-8th good, 9-12th warning, >12th bad. Method: compute from `innerText`
- [ ] Flesch Reading Ease: 60-80 good, 40-60 warning, <40 bad. Method: compute from `innerText`
- [ ] Heading frequency: every 200-300 words good, 300-600 warning, >600 bad. Method: count words between h1-h6

---

## Category 5: Layout & Responsiveness

- [ ] Content width: readable line length (45-75 characters for body text)
- [ ] Viewport fit: no horizontal scroll at the current viewport
- [ ] Element overflow: text truncates gracefully (ellipsis, not clip)
- [ ] Image sizing: images are properly constrained, no layout shift on load
- [ ] Whitespace: balanced, no cramped or excessively empty areas
- [ ] Z-index: overlapping elements stack correctly (dropdowns, modals, tooltips)

---

## Category 6: Navigation & Wayfinding

### Existing Checks

- [ ] Current location: user knows where they are (breadcrumbs, active nav state, page title)
- [ ] Back navigation: browser back button works as expected
- [ ] URL reflects state: deep-linkable, shareable
- [ ] Dead ends: no pages without a clear next action or way to navigate away
- [ ] Breadcrumbs: present on nested pages, clickable

### Enhanced Checks

- [ ] Primary nav item count: 5-7 good, 8-9 warning, >9 bad
- [ ] Dropdown item count per group: <=7 good, 8-15 warning, >15 bad
- [ ] Click depth to key pages: <=3 good, 4 warning, >5 bad
- [ ] Breadcrumbs at depth >= 3: present = good, absent = bad
- [ ] CTA count per view: 1 primary good, 2-3 warning, >3 competing bad
- [ ] Back button fidelity: 100% good, <80% bad

---

## Category 7: Forms & Input

### Existing Checks

- [ ] Validation timing: inline validation on blur, not only on submit
- [ ] Required indicators: clear marking of required fields
- [ ] Input types: correct HTML input types (email, tel, number, url)
- [ ] Autofill: standard fields work with browser autofill
- [ ] Multi-step forms: progress indicator, ability to go back
- [ ] Destructive actions: confirmation before irreversible operations

### Enhanced Checks

- [ ] Visible fields per section: 3-5 good, 6-7 warning, >7 bad
- [ ] Error message position: inline = good, top of form = warning, console/alert = bad
- [ ] Error message actionability: names field + fix = good, generic = warning, missing = bad
- [ ] Validation timing (granular): on-blur = good, on-submit only = warning, premature = bad
- [ ] Multi-step progress indicator: present for >5 fields = good, absent = bad
- [ ] Destructive action confirmation: specific verb = good, generic OK/Cancel = warning, none = bad
- [ ] Undo availability: undo toast = good, confirmation only = warning, neither = bad

---

## Category 8: Feedback & Response

### Existing Checks

- [ ] Action feedback: every user action gets visible confirmation
- [ ] Loading indicators: present during async operations, appropriate type (spinner vs skeleton vs progress)
- [ ] Optimistic updates: UI responds immediately where appropriate
- [ ] Error recovery: clear path to retry or correct after errors
- [ ] Success confirmation: user knows the action completed

### Enhanced Checks

- [ ] Skeleton screen presence: skeleton for loads >300ms = good, spinner only = warning, blank = bad
- [ ] Blank screen time: 0ms = good, any blank period = bad
- [ ] CLS during loading: 0 = good, <0.1 = warning, >0.1 = bad
- [ ] Animation duration: 200-300ms = good, 100-500ms = warning, >500ms = bad
- [ ] Toast/notification duration: 3-5s = good, <2s or no auto-dismiss for errors = bad
- [ ] Pull-to-refresh on scrollable lists: present = good, absent = bad
- [ ] Search-as-you-type latency: <200ms good, 200-500ms warning, >500ms bad

---

## Category 9: Data Display & Scalability (10 checks)

- [ ] Page scroll depth ratio: <=3x good, 3-5x warning, >5x bad. Method: `scrollHeight / clientHeight`
- [ ] Repeated item count without pagination: <=25 good, 25-50 warning, >50 bad
- [ ] Pagination controls present: present when items >25 = good, absent = bad
- [ ] Search input present: present when items >50 = good, absent = bad
- [ ] Filter controls present: present when items >25 = good, absent = bad
- [ ] Sticky header on long pages: present when scroll >3x = good, absent = bad
- [ ] Empty state quality: explanation + CTA + visual = good, text only = warning, blank = bad
- [ ] Virtual scroll for large lists: present when items >200 = good, absent = bad
- [ ] Scroll-to-action distance: CTA within 2 viewports = good, >2 = bad
- [ ] Items-per-page count: 10-50 good, 50-100 warning, >100 bad

---

## Category 10: Visual Complexity & Consistency (12 checks)

- [ ] Distinct font sizes: <=6 good, 7-9 warning, >10 bad
- [ ] Distinct font families: <=2 good, 3 warning, >3 bad
- [ ] Distinct font-size/weight combos: <=10 good, 11-15 warning, >15 bad
- [ ] Distinct colors in use: <=15 good, 16-25 warning, >25 bad
- [ ] Spacing grid conformance (4px): >90% good, 70-90% warning, <70% bad
- [ ] Alignment consistency: <=5 left-edge clusters good, 6-8 warning, >8 bad
- [ ] Visual balance (Ngo score): >0.85 good, 0.6-0.85 warning, <0.6 bad
- [ ] Content line length: 45-75 chars good, 75-90 or 30-45 warning, >90 or <30 bad
- [ ] Whitespace ratio: 30-50% good, 20-30% warning, <20% bad
- [ ] Icon consistency (stroke/fill): uniform = good, mixed = bad
- [ ] Icon sizing consistency: all same viewBox = good, multiple = bad
- [ ] Button style variations: 1-3 good, 4-5 warning, >5 bad

---

## Measurement Scripts (`browser_evaluate`)

JavaScript snippets for all automatable checks. Each returns a structured result object.

### Scroll Depth Ratio

```javascript
(() => {
  const ratio = document.documentElement.scrollHeight / document.documentElement.clientHeight;
  const grade = ratio <= 3 ? 'good' : ratio <= 5 ? 'warning' : 'bad';
  return { check: 'scroll_depth_ratio', value: Math.round(ratio * 100) / 100, grade };
})()
```

### Repeated Item Count

```javascript
(() => {
  // Heuristic: find the most common repeated class pattern among sibling elements
  const lists = document.querySelectorAll('ul, ol, [role="list"], table > tbody');
  let maxCount = 0;
  lists.forEach(list => {
    const children = list.children.length;
    if (children > maxCount) maxCount = children;
  });
  const grade = maxCount <= 25 ? 'good' : maxCount <= 50 ? 'warning' : 'bad';
  return { check: 'repeated_item_count', value: maxCount, grade };
})()
```

### Font Size / Family / Combo Audit

```javascript
(() => {
  const textEls = document.querySelectorAll('p, span, a, li, h1, h2, h3, h4, h5, h6, label, button, input, td, th, div');
  const sizes = new Set();
  const families = new Set();
  const combos = new Set();

  textEls.forEach(el => {
    const style = getComputedStyle(el);
    if (el.innerText && el.innerText.trim().length > 0) {
      sizes.add(style.fontSize);
      families.add(style.fontFamily.split(',')[0].trim().replace(/['"]/g, ''));
      combos.add(`${style.fontSize}|${style.fontWeight}|${style.fontFamily.split(',')[0].trim()}`);
    }
  });

  const sizeGrade = sizes.size <= 6 ? 'good' : sizes.size <= 9 ? 'warning' : 'bad';
  const familyGrade = families.size <= 2 ? 'good' : families.size === 3 ? 'warning' : 'bad';
  const comboGrade = combos.size <= 10 ? 'good' : combos.size <= 15 ? 'warning' : 'bad';

  return {
    check: 'font_audit',
    distinctSizes: { value: sizes.size, grade: sizeGrade, values: [...sizes] },
    distinctFamilies: { value: families.size, grade: familyGrade, values: [...families] },
    distinctCombos: { value: combos.size, grade: comboGrade }
  };
})()
```

### Spacing Grid Conformance (4px)

```javascript
(() => {
  const els = document.querySelectorAll('*');
  let total = 0;
  let conforming = 0;
  const props = ['marginTop', 'marginRight', 'marginBottom', 'marginLeft',
                 'paddingTop', 'paddingRight', 'paddingBottom', 'paddingLeft'];

  els.forEach(el => {
    const style = getComputedStyle(el);
    props.forEach(prop => {
      const val = parseFloat(style[prop]);
      if (val > 0) {
        total++;
        if (val % 4 === 0) conforming++;
      }
    });
  });

  const pct = total > 0 ? Math.round((conforming / total) * 100) : 100;
  const grade = pct > 90 ? 'good' : pct >= 70 ? 'warning' : 'bad';
  return { check: 'spacing_grid_conformance', conformingPct: pct, total, conforming, grade };
})()
```

### Alignment Clustering

```javascript
(() => {
  const els = document.querySelectorAll('h1, h2, h3, h4, h5, h6, p, div, section, article, li, span, a, button, input, img');
  const lefts = [];
  const tolerance = 2; // px

  els.forEach(el => {
    const rect = el.getBoundingClientRect();
    if (rect.width > 0 && rect.height > 0 && rect.top < window.innerHeight) {
      lefts.push(Math.round(rect.left));
    }
  });

  // Cluster left edges within tolerance
  const sorted = [...lefts].sort((a, b) => a - b);
  const clusters = [];
  let current = null;

  sorted.forEach(val => {
    if (current === null || val - current > tolerance) {
      clusters.push(val);
      current = val;
    }
  });

  const grade = clusters.length <= 5 ? 'good' : clusters.length <= 8 ? 'warning' : 'bad';
  return { check: 'alignment_clustering', clusterCount: clusters.length, clusters, grade };
})()
```

### Visual Balance (Ngo Formula)

```javascript
(() => {
  // Ngo visual balance: BM = (BM_h + BM_v) / 2
  // BM_h = 1 - |W_left - W_right| / max(W_left + W_right, 1)
  // BM_v = 1 - |W_top - W_bottom| / max(W_top + W_bottom, 1)
  // W = sum of (element area) weighted by distance from center axis

  const vw = window.innerWidth;
  const vh = window.innerHeight;
  const cx = vw / 2;
  const cy = vh / 2;

  let wLeft = 0, wRight = 0, wTop = 0, wBottom = 0;

  const els = document.querySelectorAll('img, svg, button, input, h1, h2, h3, h4, h5, h6, p, a, video, canvas, [role="img"]');
  els.forEach(el => {
    const rect = el.getBoundingClientRect();
    if (rect.width === 0 || rect.height === 0) return;
    const area = rect.width * rect.height;
    const elCx = rect.left + rect.width / 2;
    const elCy = rect.top + rect.height / 2;

    if (elCx < cx) wLeft += area;
    else wRight += area;

    if (elCy < cy) wTop += area;
    else wBottom += area;
  });

  const bmH = 1 - Math.abs(wLeft - wRight) / Math.max(wLeft + wRight, 1);
  const bmV = 1 - Math.abs(wTop - wBottom) / Math.max(wTop + wBottom, 1);
  const bm = (bmH + bmV) / 2;
  const score = Math.round(bm * 100) / 100;

  const grade = score > 0.85 ? 'good' : score >= 0.6 ? 'warning' : 'bad';
  return { check: 'visual_balance_ngo', score, bmH: Math.round(bmH * 100) / 100, bmV: Math.round(bmV * 100) / 100, grade };
})()
```

### Whitespace Ratio

```javascript
(() => {
  const vw = window.innerWidth;
  const vh = window.innerHeight;
  const viewportArea = vw * vh;
  let occupiedArea = 0;

  const els = document.querySelectorAll('img, svg, button, input, textarea, select, video, canvas, table, h1, h2, h3, h4, h5, h6, p, a, span, li, label');
  els.forEach(el => {
    const rect = el.getBoundingClientRect();
    if (rect.width > 0 && rect.height > 0 &&
        rect.bottom > 0 && rect.top < vh &&
        rect.right > 0 && rect.left < vw) {
      const visW = Math.min(rect.right, vw) - Math.max(rect.left, 0);
      const visH = Math.min(rect.bottom, vh) - Math.max(rect.top, 0);
      occupiedArea += visW * visH;
    }
  });

  // Clamp to viewport (overlapping elements may exceed 100%)
  const ratio = Math.min(occupiedArea / viewportArea, 1);
  const whitespace = Math.round((1 - ratio) * 100);
  const grade = whitespace >= 30 && whitespace <= 50 ? 'good' : whitespace >= 20 ? 'warning' : 'bad';
  return { check: 'whitespace_ratio', whitespacePct: whitespace, grade };
})()
```

### Flesch-Kincaid Formulas

```javascript
(() => {
  // Syllable estimation heuristic
  function countSyllables(word) {
    word = word.toLowerCase().replace(/[^a-z]/g, '');
    if (word.length <= 2) return 1;
    word = word.replace(/(?:[^laeiouy]es|ed|[^laeiouy]e)$/, '');
    word = word.replace(/^y/, '');
    const matches = word.match(/[aeiouy]{1,2}/g);
    return matches ? matches.length : 1;
  }

  const text = document.body.innerText || '';
  const words = text.split(/\s+/).filter(w => w.length > 0);
  const sentences = text.split(/[.!?]+/).filter(s => s.trim().length > 0);

  const wordCount = words.length;
  const sentenceCount = Math.max(sentences.length, 1);
  let syllableCount = 0;
  words.forEach(w => { syllableCount += countSyllables(w); });

  const wordsPerSentence = wordCount / sentenceCount;
  const syllablesPerWord = syllableCount / Math.max(wordCount, 1);

  // Grade Level: 0.39 * (words/sentences) + 11.8 * (syllables/words) - 15.59
  const gradeLevel = 0.39 * wordsPerSentence + 11.8 * syllablesPerWord - 15.59;

  // Reading Ease: 206.835 - 1.015 * (words/sentences) - 84.6 * (syllables/words)
  const readingEase = 206.835 - 1.015 * wordsPerSentence - 84.6 * syllablesPerWord;

  const glRounded = Math.round(gradeLevel * 10) / 10;
  const reRounded = Math.round(readingEase * 10) / 10;

  const glGrade = glRounded <= 8 ? 'good' : glRounded <= 12 ? 'warning' : 'bad';
  const reGrade = reRounded >= 60 && reRounded <= 80 ? 'good' : reRounded >= 40 ? 'warning' : 'bad';

  return {
    check: 'flesch_kincaid',
    gradeLevel: { value: glRounded, grade: glGrade },
    readingEase: { value: reRounded, grade: reGrade },
    stats: { words: wordCount, sentences: sentenceCount, syllables: syllableCount }
  };
})()
```

### Heading Frequency

```javascript
(() => {
  const body = document.body;
  const walker = document.createTreeWalker(body, NodeFilter.SHOW_ELEMENT | NodeFilter.SHOW_TEXT);
  const headingTags = new Set(['H1', 'H2', 'H3', 'H4', 'H5', 'H6']);
  let wordsBetween = 0;
  const gaps = [];

  while (walker.nextNode()) {
    const node = walker.currentNode;
    if (node.nodeType === Node.ELEMENT_NODE && headingTags.has(node.tagName)) {
      if (wordsBetween > 0) gaps.push(wordsBetween);
      wordsBetween = 0;
    } else if (node.nodeType === Node.TEXT_NODE) {
      const words = node.textContent.trim().split(/\s+/).filter(w => w.length > 0);
      wordsBetween += words.length;
    }
  }
  if (wordsBetween > 0) gaps.push(wordsBetween);

  const maxGap = gaps.length > 0 ? Math.max(...gaps) : 0;
  const avgGap = gaps.length > 0 ? Math.round(gaps.reduce((a, b) => a + b, 0) / gaps.length) : 0;
  const grade = maxGap <= 300 ? 'good' : maxGap <= 600 ? 'warning' : 'bad';

  return { check: 'heading_frequency', maxGap, avgGap, gapCount: gaps.length, grade };
})()
```

### Information Density (Words in Viewport)

```javascript
(() => {
  const vh = window.innerHeight;
  const els = document.querySelectorAll('p, span, a, li, h1, h2, h3, h4, h5, h6, label, td, th, button, div');
  let wordCount = 0;
  const counted = new Set();

  els.forEach(el => {
    const rect = el.getBoundingClientRect();
    if (rect.top < vh && rect.bottom > 0 && !counted.has(el)) {
      // Only count leaf text to avoid double counting
      if (el.children.length === 0 || el.tagName === 'P' || el.tagName === 'LI') {
        const text = el.innerText || '';
        wordCount += text.split(/\s+/).filter(w => w.length > 0).length;
        counted.add(el);
      }
    }
  });

  const grade = wordCount >= 150 && wordCount <= 300 ? 'good' : wordCount <= 500 ? 'warning' : 'bad';
  return { check: 'information_density', wordsInViewport: wordCount, grade };
})()
```

### Nav Item Count, Dropdown Count, CTA Count

```javascript
(() => {
  // Primary nav items
  const navEls = document.querySelectorAll('nav > ul > li, nav > a, [role="navigation"] > ul > li');
  const navCount = navEls.length;
  const navGrade = navCount <= 7 ? 'good' : navCount <= 9 ? 'warning' : 'bad';

  // Dropdown item counts
  const dropdowns = document.querySelectorAll('[role="menu"], select, [class*="dropdown"] ul');
  let maxDropdownItems = 0;
  dropdowns.forEach(dd => {
    const items = dd.querySelectorAll('[role="menuitem"], option, li');
    if (items.length > maxDropdownItems) maxDropdownItems = items.length;
  });
  const ddGrade = maxDropdownItems <= 7 ? 'good' : maxDropdownItems <= 15 ? 'warning' : 'bad';

  // CTA count — primary-styled buttons in viewport
  const buttons = document.querySelectorAll('button, a[role="button"], [class*="btn-primary"], [class*="cta"]');
  let ctaCount = 0;
  buttons.forEach(btn => {
    const rect = btn.getBoundingClientRect();
    if (rect.top < window.innerHeight && rect.bottom > 0 && rect.width > 0) {
      const style = getComputedStyle(btn);
      // Heuristic: prominent buttons have a non-transparent background
      if (style.backgroundColor !== 'rgba(0, 0, 0, 0)' && style.backgroundColor !== 'transparent') {
        ctaCount++;
      }
    }
  });
  const ctaGrade = ctaCount === 1 ? 'good' : ctaCount <= 3 ? 'warning' : 'bad';

  return {
    check: 'nav_and_cta_audit',
    navItems: { count: navCount, grade: navGrade },
    maxDropdownItems: { count: maxDropdownItems, grade: ddGrade },
    ctaCount: { count: ctaCount, grade: ctaGrade }
  };
})()
```

### DOM Element Count

```javascript
(() => {
  const count = document.querySelectorAll('*').length;
  const grade = count < 1500 ? 'good' : count <= 3000 ? 'warning' : 'bad';
  return { check: 'dom_element_count', value: count, grade };
})()
```

### Color Audit

```javascript
(() => {
  const els = document.querySelectorAll('*');
  const colors = new Set();

  els.forEach(el => {
    const style = getComputedStyle(el);
    if (el.innerText && el.innerText.trim().length > 0) {
      colors.add(style.color);
    }
    if (style.backgroundColor !== 'rgba(0, 0, 0, 0)' && style.backgroundColor !== 'transparent') {
      colors.add(style.backgroundColor);
    }
  });

  const grade = colors.size <= 15 ? 'good' : colors.size <= 25 ? 'warning' : 'bad';
  return { check: 'color_audit', distinctColors: colors.size, grade, values: [...colors] };
})()
```

---

## Severity Grading Criteria

Each category receives a severity grade based on the number and impact of failed checks:

| Grade | Criteria |
|-------|----------|
| **PASS** | All checks in category pass |
| **MINOR** | 1-2 checks fail, low impact |
| **MAJOR** | 3+ checks fail, or any high-impact check fails |
| **CRITICAL** | Fundamental usability broken (e.g., no pagination on 100+ items, >10x scroll depth, zero accessibility, no error recovery) |

### High-Impact Checks (trigger MAJOR on single failure)

- Color contrast below WCAG AA (Category 4)
- No form labels (Category 4)
- No loading indicators on async operations (Category 8)
- No pagination on 50+ items (Category 9)
- Scroll depth >5x viewport with no sticky navigation (Category 9)
- >10 distinct font sizes (Category 10)

---

## Binary Scorecard Rules

Each check maps to **Pass (1)** or **Fail (0)**. The total per screen is **X/75**.

For checks with three-tier thresholds (good / warning / bad):
- **good** = Pass (1)
- **warning** = Pass (1), but noted as a finding with MINOR severity
- **bad** = Fail (0)

### Per-Category Check Counts

| Category | Checks |
|----------|--------|
| 1. Visual Consistency | 7 |
| 2. Component States | 8 |
| 3. Copy & Microcopy | 7 |
| 4. Accessibility | 13 |
| 5. Layout & Responsiveness | 6 |
| 6. Navigation & Wayfinding | 11 |
| 7. Forms & Input | 13 |
| 8. Feedback & Response | 12 |
| 9. Data Display & Scalability | 10 |
| 10. Visual Complexity & Consistency | 12 |
| **Total** | **99** |

> **Note:** Not all checks apply to every screen. The denominator adjusts to only count applicable checks. The target total of ~75 reflects a typical screen; screens with no forms, for example, would exclude Category 7 enhanced checks.

### Output Template

```markdown
## UX Audit Results

### Scorecard: X/Y Pass (Z%)

### [Screen Name] — [URL]

| Category | Grade | Pass/Total | Findings |
|----------|-------|------------|----------|
| Visual Consistency | MINOR | 6/7 | 1 finding |
| Component States | PASS | 8/8 | -- |
| Copy & Microcopy | PASS | 7/7 | -- |
| Accessibility | MINOR | 11/13 | 2 findings |
| Layout & Responsiveness | PASS | 6/6 | -- |
| Navigation & Wayfinding | MAJOR | 7/11 | 4 findings |
| Forms & Input | MINOR | 11/13 | 2 findings |
| Feedback & Response | PASS | 12/12 | -- |
| Data Display & Scalability | CRITICAL | 3/10 | 7 findings |
| Visual Complexity & Consistency | MINOR | 10/12 | 2 findings |

### Findings Detail
1. [MAJOR] **Finding title** — Description of what is wrong, where, and what it should be.
2. [MINOR] **Finding title** — Description with specific measurements.
3. ...
```
