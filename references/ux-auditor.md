# UX Auditor — Complete Rubric Reference

This reference contains all 10 audit categories, measurement scripts, thresholds, and grading criteria for the UX auditor agent. It covers ~75 checks across Visual Consistency, Component States, Copy & Microcopy, Accessibility, Layout & Responsiveness, Navigation & Wayfinding, Forms & Input, Feedback & Response, Data Display & Scalability, and Visual Complexity & Consistency.

Each check maps to a binary Pass (1) or Fail (0). The total score per screen is X/75. Severity grades (PASS / MINOR / MAJOR / CRITICAL) are assigned per category based on the number and impact of failures.

---

## Measurement Tier Legend

Each check is tagged with its measurement confidence level:

- **`[D]` Deterministic** — Fully measurable via `browser_evaluate`. Returns a numeric value with a clear threshold. Same page always produces the same result. High confidence.
- **`[H]` Heuristic** — Measurable via `browser_evaluate` but with known false positive/negative risks (<5% error rate), OR requires Playwright interaction sequence. Reliable signal, not definitive.
- **`[J]` LLM-Judgment** — Requires visual interpretation or semantic understanding. A programmatic pre-filter narrows what the LLM evaluates by 75-85%. Lower confidence.

### Threshold Citation Legend

- **`[research]`** — Backed by peer-reviewed research or official standards (WCAG, Apple HIG, MD3, Google Web Vitals)
- **`[convention]`** — Widely accepted industry convention
- **`[heuristic]`** — Team-chosen threshold, reasonable but not externally validated

---

## Measurement Utilities

### Shadow DOM Traversal

Many modern web apps use Shadow DOM (Web Components, Shoelace, Lit, Ionic). Standard `querySelectorAll` does not traverse shadow roots. Use this utility in scripts that need complete DOM coverage:

```javascript
function deepQuerySelectorAll(selector, root = document) {
  const results = [...root.querySelectorAll(selector)];
  root.querySelectorAll('*').forEach(el => {
    if (el.shadowRoot) {
      results.push(...deepQuerySelectorAll(selector, el.shadowRoot));
    }
  });
  return results;
}
```

Use `deepQuerySelectorAll` instead of `document.querySelectorAll` in scripts that measure touch targets, form attributes, text elements, interactive elements, or any check where Shadow DOM elements would be missed. For performance, only use when `document.querySelectorAll('*').length` and custom element detection suggest shadow roots are present.

**Known limitation:** Cross-origin iframes cannot be traversed. Third-party widgets (chat, payments, auth) inside cross-origin iframes are invisible to all measurement scripts. Document this in findings as "iframe content not audited."

---

## Category 1: Visual Consistency

- [ ] `[H]` Typography: font sizes, weights, and line heights follow a consistent scale
- [ ] `[H]` Spacing: padding and margins use a consistent system (4px/8px grid or similar)
- [ ] `[H]` Colors: brand colors are used consistently, no off-by-one hex values
- [ ] `[H]` Border radii: consistent across similar elements (buttons, cards, inputs)
- [ ] `[H]` Shadows: consistent depth system, not arbitrary values
- [ ] `[J]` Icons: consistent style (outline vs filled), consistent sizing
- [ ] `[H]` Alignment: elements are properly aligned to a grid, no off-by-1px misalignment `[heuristic]`

---

## Category 2: Component States

- [ ] `[J]` Default state: clear, not ambiguous
- [ ] `[H]` Hover state: present on all interactive elements, provides visual feedback
- [ ] `[D]` Focus state: visible focus ring for keyboard navigation (accessibility)
- [ ] `[J]` Active/pressed state: provides tactile feedback
- [ ] `[D]` Disabled state: visually distinct, not clickable
- [ ] `[H]` Loading state: present where async operations occur, uses consistent pattern
- [ ] `[J]` Empty state: helpful message and action when no data exists (not just blank space)
- [ ] `[J]` Error state: clear, specific, actionable error messages near the relevant field

---

## Category 3: Copy & Microcopy

- [ ] `[H]` Error messages: specific ("Email is already registered") not vague ("Something went wrong")
- [ ] `[H]` Button labels: action-oriented ("Save Changes" not "Submit"), consistent capitalization
- [ ] `[H]` Placeholder text: helpful examples, not labels (labels should be above the field)
- [ ] `[H]` Confirmation messages: tell the user what happened ("Profile updated" not "Success")
- [ ] `[H]` Empty states: explain what goes here and how to add content
- [ ] `[J]` Tooltips: present where needed, concise, not redundant with visible labels
- [ ] `[H]` Grammar and spelling: no typos, consistent voice and tense

---

## Category 4: Accessibility

### Existing Checks

- [ ] `[D]` Color contrast: text meets WCAG AA (4.5:1 for normal text, 3:1 for large) `[research: WCAG 2.1]`
- [ ] `[D]` Touch targets: at least 44x44px on interactive elements `[research: WCAG 2.1]`
- [ ] `[D]` Form labels: every input has an associated label (not just placeholder)
- [ ] `[D]` Alt text: images have meaningful alt text (or empty alt for decorative)
- [ ] `[D]` Heading hierarchy: h1 -> h2 -> h3, no skipped levels `[research: WCAG 2.1]`
- [ ] `[D]` Tab order: logical, follows visual flow
- [ ] `[H]` Screen reader: critical content is not conveyed by color alone

### Cognitive Load Checks (Enhanced)

- [ ] `[D]` Information density (words/viewport): 150-300 good, 300-500 warning, >500 bad. Method: `innerText` word count within viewport bounds `[heuristic]`
- [ ] `[D]` DOM element count: <1500 good, 1500-3000 warning, >3000 bad. Method: `querySelectorAll('*').length`
- [ ] `[H]` Choices per interaction context: <=5 good, 6-9 warning, >9 bad. Method: count visible actionable elements (buttons/links) in focused section
- [ ] `[D]` Flesch-Kincaid grade level: 7-8th good, 9-12th warning, >12th bad. Method: compute from `innerText`
- [ ] `[D]` Flesch Reading Ease: 60-80 good, 40-60 warning, <40 bad. Method: compute from `innerText`
- [ ] `[D]` Heading frequency: every 200-300 words good, 300-600 warning, >600 bad. Method: count words between h1-h6 `[heuristic]`

---

## Category 5: Layout & Responsiveness

- [ ] `[D]` Content width: readable line length (45-75 characters for body text) `[research: WCAG 2.1]`
- [ ] `[D]` Viewport fit: no horizontal scroll at the current viewport
- [ ] `[D]` Element overflow: text truncates gracefully (ellipsis, not clip)
- [ ] `[H]` Image sizing: images are properly constrained, no layout shift on load
- [ ] `[J]` Whitespace: balanced, no cramped or excessively empty areas `[heuristic]`
- [ ] `[J]` Z-index: overlapping elements stack correctly (dropdowns, modals, tooltips)

---

## Category 6: Navigation & Wayfinding

### Existing Checks

- [ ] `[D]` Current location: user knows where they are (breadcrumbs, active nav state, page title)
- [ ] `[H]` Back navigation: browser back button works as expected
- [ ] `[H]` URL reflects state: deep-linkable, shareable
- [ ] `[H]` Dead ends: no pages without a clear next action or way to navigate away
- [ ] `[D]` Breadcrumbs: present on nested pages, clickable

### Enhanced Checks

- [ ] `[D]` Primary nav item count: 5-7 good, 8-9 warning, >9 bad `[convention]`
- [ ] `[D]` Dropdown item count per group: <=7 good, 8-15 warning, >15 bad
- [ ] `[H]` Click depth to key pages: <=3 good, 4 warning, >5 bad
- [ ] `[D]` Breadcrumbs at depth >= 3: present = good, absent = bad
- [ ] `[D]` CTA count per view: 1 primary good, 2-3 warning, >3 competing bad `[convention]`
- [ ] `[H]` Back button fidelity: 100% good, <80% bad

---

## Category 7: Forms & Input

### Existing Checks

- [ ] `[H]` Validation timing: inline validation on blur, not only on submit
- [ ] `[D]` Required indicators: clear marking of required fields
- [ ] `[D]` Input types: correct HTML input types (email, tel, number, url)
- [ ] `[D]` Autofill: standard fields work with browser autofill
- [ ] `[D]` Multi-step forms: progress indicator, ability to go back
- [ ] `[H]` Destructive actions: confirmation before irreversible operations

### Enhanced Checks

- [ ] `[D]` Visible fields per section: 3-5 good, 6-7 warning, >7 bad
- [ ] `[D]` Error message position: inline = good, top of form = warning, console/alert = bad
- [ ] `[J]` Error message actionability: names field + fix = good, generic = warning, missing = bad
- [ ] `[H]` Validation timing (granular): on-blur = good, on-submit only = warning, premature = bad
- [ ] `[D]` Multi-step progress indicator: present for >5 fields = good, absent = bad
- [ ] `[D]` Destructive action confirmation: specific verb = good, generic OK/Cancel = warning, none = bad
- [ ] `[J]` Undo availability: undo toast = good, confirmation only = warning, neither = bad

---

## Category 8: Feedback & Response

### Existing Checks

- [ ] `[J]` Action feedback: every user action gets visible confirmation
- [ ] `[H]` Loading indicators: present during async operations, appropriate type (spinner vs skeleton vs progress)
- [ ] `[J]` Optimistic updates: UI responds immediately where appropriate
- [ ] `[H]` Error recovery: clear path to retry or correct after errors
- [ ] `[H]` Success confirmation: user knows the action completed

### Enhanced Checks

- [ ] `[D]` Skeleton screen presence: skeleton for loads >300ms = good, spinner only = warning, blank = bad
- [ ] `[H]` Blank screen time: 0ms = good, any blank period = bad
- [ ] `[D]` CLS during loading: 0 = good, <0.1 = warning, >0.1 = bad `[research: Google Web Vitals]`
- [ ] `[D]` Animation duration: 200-300ms = good, 100-500ms = warning, >500ms = bad `[convention]`
- [ ] `[H]` Toast/notification duration: 3-5s = good, <2s or no auto-dismiss for errors = bad
- [ ] `[H]` Pull-to-refresh on scrollable lists: present = good, absent = bad
- [ ] `[H]` Search-as-you-type latency: <200ms good, 200-500ms warning, >500ms bad

---

## Category 9: Data Display & Scalability (10 checks)

- [ ] `[D]` Page scroll depth ratio: <=3x good, 3-5x warning, >5x bad. Method: `scrollHeight / clientHeight` `[heuristic]`
- [ ] `[D]` Repeated item count without pagination: <=25 good, 25-50 warning, >50 bad
- [ ] `[D]` Pagination controls present: present when items >25 = good, absent = bad
- [ ] `[D]` Search input present: present when items >50 = good, absent = bad
- [ ] `[D]` Filter controls present: present when items >25 = good, absent = bad
- [ ] `[D]` Sticky header on long pages: present when scroll >3x = good, absent = bad
- [ ] `[J]` Empty state quality: explanation + CTA + visual = good, text only = warning, blank = bad
- [ ] `[H]` Virtual scroll for large lists: present when items >200 = good, absent = bad
- [ ] `[D]` Scroll-to-action distance: CTA within 2 viewports = good, >2 = bad
- [ ] `[D]` Items-per-page count: 10-50 good, 50-100 warning, >100 bad

---

## Category 10: Visual Complexity & Consistency (12 checks)

- [ ] `[D]` Distinct font sizes: <=6 good, 7-9 warning, >10 bad `[convention]`
- [ ] `[D]` Distinct font families: <=2 good, 3 warning, >3 bad
- [ ] `[D]` Distinct font-size/weight combos: <=10 good, 11-15 warning, >15 bad
- [ ] `[D]` Distinct colors in use: <=15 good, 16-25 warning, >25 bad
- [ ] `[D]` Spacing grid conformance (4px): >90% good, 70-90% warning, <70% bad
- [ ] `[H]` Alignment consistency: <=5 left-edge clusters good, 6-8 warning, >8 bad `[heuristic]`
- [ ] `[H]` Visual balance (Ngo score): >0.85 good, 0.6-0.85 warning, <0.6 bad
- [ ] `[D]` Content line length: 45-75 chars good, 75-90 or 30-45 warning, >90 or <30 bad `[research: WCAG 2.1]`
- [ ] `[H]` Whitespace ratio: 30-50% good, 20-30% warning, <20% bad `[heuristic]`
- [ ] `[J]` Icon consistency (stroke/fill): uniform = good, mixed = bad
- [ ] `[D]` Icon sizing consistency: all same viewBox = good, multiple = bad
- [ ] `[H]` Button style variations: 1-3 good, 4-5 warning, >5 bad `[convention]`

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
  // Check ul/ol/table containers
  const lists = document.querySelectorAll('ul, ol, [role="list"], table > tbody');
  let maxCount = 0;
  let maxSource = 'none';
  lists.forEach(list => {
    const children = list.children.length;
    if (children > maxCount) {
      maxCount = children;
      maxSource = list.tagName.toLowerCase();
    }
  });

  // Also find containers whose children share a common class pattern
  // (>= 50% of children with same class list)
  const containers = document.querySelectorAll('div, section, main, article');
  containers.forEach(container => {
    const children = Array.from(container.children);
    if (children.length < 3) return;
    const classCounts = {};
    children.forEach(child => {
      const key = Array.from(child.classList).sort().join(' ');
      if (key) classCounts[key] = (classCounts[key] || 0) + 1;
    });
    const maxClassCount = Math.max(...Object.values(classCounts), 0);
    if (maxClassCount >= children.length * 0.5 && maxClassCount > maxCount) {
      maxCount = maxClassCount;
      maxSource = 'class-pattern';
    }
  });

  const grade = maxCount <= 25 ? 'good' : maxCount <= 50 ? 'warning' : 'bad';
  return { check: 'repeated_item_count', value: maxCount, source: maxSource, grade };
})()
```

### Font Size / Family / Combo Audit

```javascript
(() => {
  const sizes = new Set();
  const families = new Set();
  const combos = new Set();

  // Use TreeWalker with SHOW_TEXT to find leaf text nodes
  const walker = document.createTreeWalker(document.body, NodeFilter.SHOW_TEXT, {
    acceptNode(node) {
      if (!node.textContent || node.textContent.trim().length === 0) return NodeFilter.FILTER_REJECT;
      return NodeFilter.FILTER_ACCEPT;
    }
  });

  while (walker.nextNode()) {
    const textNode = walker.currentNode;
    const el = textNode.parentElement;
    if (!el) continue;
    const rect = el.getBoundingClientRect();
    if (rect.width === 0 && rect.height === 0) continue;
    const style = getComputedStyle(el);
    const rawSize = parseFloat(style.fontSize);
    const roundedSize = Math.round(rawSize * 10) / 10 + 'px';
    sizes.add(roundedSize);
    families.add(style.fontFamily.split(',')[0].trim().replace(/['"]/g, ''));
    combos.add(`${roundedSize}|${style.fontWeight}|${style.fontFamily.split(',')[0].trim()}`);
  }

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
    if (style.display === 'none') return;
    props.forEach(prop => {
      const raw = parseFloat(style[prop]);
      const val = Math.abs(raw);
      if (val > 0) {
        total++;
        if (Math.round(val) % 4 === 0) conforming++;
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
  const blockDisplays = new Set(['block', 'flex', 'grid', 'table', 'list-item', 'table-row', 'table-cell']);
  const els = document.querySelectorAll('h1, h2, h3, h4, h5, h6, p, div, section, article, li, button, input, img, nav, main, aside, footer, header, form, ul, ol');
  const lefts = [];
  const tolerance = 2; // px

  els.forEach(el => {
    const style = getComputedStyle(el);
    if (!blockDisplays.has(style.display)) return;
    const rect = el.getBoundingClientRect();
    if (rect.width > 0 && rect.height > 0 && rect.top < window.innerHeight && rect.bottom > 0) {
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
  // Fix: process deepest-first, skip parents whose descendants were counted, clip to viewport

  const vw = window.innerWidth;
  const vh = window.innerHeight;
  const cx = vw / 2;
  const cy = vh / 2;

  let wLeft = 0, wRight = 0, wTop = 0, wBottom = 0;
  const counted = new Set();

  const els = Array.from(document.querySelectorAll('img, svg, button, input, h1, h2, h3, h4, h5, h6, p, a, video, canvas, [role="img"]'));
  // Sort deepest-first: elements with greater depth processed first
  els.sort((a, b) => {
    let dA = 0, dB = 0;
    let n = a; while (n) { dA++; n = n.parentElement; }
    n = b; while (n) { dB++; n = n.parentElement; }
    return dB - dA;
  });

  els.forEach(el => {
    const rect = el.getBoundingClientRect();
    if (rect.width === 0 || rect.height === 0) return;
    // Clip to viewport bounds
    if (rect.bottom <= 0 || rect.top >= vh || rect.right <= 0 || rect.left >= vw) return;
    // Skip if a descendant was already counted (parent would double-count)
    let dominated = false;
    counted.forEach(c => { if (el.contains(c)) dominated = true; });
    if (dominated) return;
    counted.add(el);

    const clippedLeft = Math.max(rect.left, 0);
    const clippedRight = Math.min(rect.right, vw);
    const clippedTop = Math.max(rect.top, 0);
    const clippedBottom = Math.min(rect.bottom, vh);
    const area = (clippedRight - clippedLeft) * (clippedBottom - clippedTop);
    const elCx = (clippedLeft + clippedRight) / 2;
    const elCy = (clippedTop + clippedBottom) / 2;

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
  // Pixel-sampling approach using document.elementFromPoint on a 20px grid
  // Eliminates double-counting from overlapping/nested elements
  const vw = window.innerWidth;
  const vh = window.innerHeight;
  const step = 20;
  let totalSamples = 0;
  let occupiedSamples = 0;

  for (let y = step / 2; y < vh; y += step) {
    for (let x = step / 2; x < vw; x += step) {
      totalSamples++;
      const el = document.elementFromPoint(x, y);
      if (el && el !== document.documentElement && el !== document.body) {
        occupiedSamples++;
      }
    }
  }

  const whitespace = totalSamples > 0 ? Math.round((1 - occupiedSamples / totalSamples) * 100) : 50;
  const grade = whitespace >= 30 && whitespace <= 50 ? 'good' : whitespace >= 20 ? 'warning' : 'bad';
  return { check: 'whitespace_ratio', whitespacePct: whitespace, totalSamples, occupiedSamples, grade };
})()
```

### Flesch-Kincaid Formulas

```javascript
(() => {
  // Language check — FK is only valid for English
  const lang = (document.documentElement.lang || '').toLowerCase();
  if (lang && !lang.startsWith('en')) {
    return { check: 'flesch_kincaid', available: false, reason: 'non-English content', detectedLang: lang };
  }

  // Syllable estimation heuristic
  function countSyllables(word) {
    word = word.toLowerCase().replace(/[^a-z]/g, '');
    if (word.length <= 2) return 1;
    word = word.replace(/(?:[^laeiouy]es|ed|[^laeiouy]e)$/, '');
    word = word.replace(/^y/, '');
    const matches = word.match(/[aeiouy]{1,2}/g);
    return matches ? matches.length : 1;
  }

  // Extract text from main content area only
  const contentEl = document.querySelector('main, article, [role="main"]') || document.body;
  const text = contentEl.innerText || '';
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
    available: true,
    contentSource: contentEl.tagName.toLowerCase(),
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
  const hiddenTags = new Set(['SCRIPT', 'STYLE', 'TEMPLATE']);
  const headingTags = new Set(['H1', 'H2', 'H3', 'H4', 'H5', 'H6']);

  const walker = document.createTreeWalker(body, NodeFilter.SHOW_ELEMENT | NodeFilter.SHOW_TEXT, {
    acceptNode(node) {
      if (node.nodeType === Node.ELEMENT_NODE) {
        if (hiddenTags.has(node.tagName)) return NodeFilter.FILTER_REJECT;
        if (getComputedStyle(node).display === 'none') return NodeFilter.FILTER_REJECT;
        return NodeFilter.FILTER_ACCEPT;
      }
      if (node.nodeType === Node.TEXT_NODE) {
        const parent = node.parentElement;
        if (parent && hiddenTags.has(parent.tagName)) return NodeFilter.FILTER_REJECT;
        return NodeFilter.FILTER_ACCEPT;
      }
      return NodeFilter.FILTER_ACCEPT;
    }
  });

  let wordsBetween = 0;
  const gaps = [];
  const headingTexts = [];

  while (walker.nextNode()) {
    const node = walker.currentNode;
    if (node.nodeType === Node.ELEMENT_NODE && headingTags.has(node.tagName)) {
      if (wordsBetween > 0) gaps.push(wordsBetween);
      wordsBetween = 0;
      headingTexts.push(node.textContent.trim());
    } else if (node.nodeType === Node.TEXT_NODE) {
      const words = node.textContent.trim().split(/\s+/).filter(w => w.length > 0);
      wordsBetween += words.length;
    }
  }
  if (wordsBetween > 0) gaps.push(wordsBetween);

  const maxGap = gaps.length > 0 ? Math.max(...gaps) : 0;
  const avgGap = gaps.length > 0 ? Math.round(gaps.reduce((a, b) => a + b, 0) / gaps.length) : 0;
  const grade = maxGap <= 300 ? 'good' : maxGap <= 600 ? 'warning' : 'bad';

  // Compound condition: detect duplicate heading texts
  const uniqueHeadings = new Set(headingTexts);
  const allIdentical = headingTexts.length > 1 && uniqueHeadings.size === 1;

  return { check: 'heading_frequency', maxGap, avgGap, gapCount: gaps.length, grade, headingCount: headingTexts.length, allIdentical };
})()
```

### Information Density (Words in Viewport)

```javascript
(() => {
  const vh = window.innerHeight;
  const vw = window.innerWidth;
  let wordCount = 0;

  // Use TreeWalker with SHOW_TEXT to count words from text nodes only
  // Eliminates double-counting from nested elements
  const walker = document.createTreeWalker(document.body, NodeFilter.SHOW_TEXT, {
    acceptNode(node) {
      if (!node.textContent || node.textContent.trim().length === 0) return NodeFilter.FILTER_REJECT;
      const el = node.parentElement;
      if (!el) return NodeFilter.FILTER_REJECT;
      const rect = el.getBoundingClientRect();
      // Check parent element is within viewport bounds
      if (rect.bottom <= 0 || rect.top >= vh || rect.right <= 0 || rect.left >= vw) return NodeFilter.FILTER_REJECT;
      if (rect.width === 0 && rect.height === 0) return NodeFilter.FILTER_REJECT;
      return NodeFilter.FILTER_ACCEPT;
    }
  });

  while (walker.nextNode()) {
    const words = walker.currentNode.textContent.trim().split(/\s+/).filter(w => w.length > 0);
    wordCount += words.length;
  }

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

  // CTA count — exclude buttons inside nav, footer, header
  const excludeContainers = new Set(['NAV', 'FOOTER', 'HEADER']);
  function isInExcludedContainer(el) {
    let parent = el.parentElement;
    while (parent) {
      if (excludeContainers.has(parent.tagName)) return true;
      parent = parent.parentElement;
    }
    return false;
  }

  const buttons = document.querySelectorAll('button, a[role="button"], [class*="btn-primary"], [class*="cta"], input[type="submit"]');
  let ctaCount = 0;
  buttons.forEach(btn => {
    const rect = btn.getBoundingClientRect();
    if (rect.top >= window.innerHeight || rect.bottom <= 0 || rect.width === 0) return;

    const cls = (btn.className || '').toLowerCase();
    const isPrimaryClass = /primary|cta/.test(cls);
    const isSubmit = btn.type === 'submit';
    const inExcluded = isInExcludedContainer(btn);

    if (inExcluded && !isPrimaryClass && !isSubmit) return;

    if (isPrimaryClass || isSubmit) {
      ctaCount++;
      return;
    }

    // Heuristic: non-transparent background + padding >= 12px + not in nav/footer
    if (!inExcluded) {
      const style = getComputedStyle(btn);
      const bg = style.backgroundColor;
      const hasBg = bg !== 'rgba(0, 0, 0, 0)' && bg !== 'transparent';
      const padL = parseFloat(style.paddingLeft) || 0;
      const padR = parseFloat(style.paddingRight) || 0;
      if (hasBg && (padL >= 12 || padR >= 12)) {
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
  // Parse CSS color string to {r, g, b} tuple
  function parseColor(str) {
    const match = str.match(/rgba?\((\d+),\s*(\d+),\s*(\d+)/);
    if (match) return { r: +match[1], g: +match[2], b: +match[3] };
    return null;
  }

  // Convert sRGB to CIELAB for perceptual distance
  function srgbToLab(c) {
    let r = c.r / 255, g = c.g / 255, b = c.b / 255;
    r = r > 0.04045 ? Math.pow((r + 0.055) / 1.055, 2.4) : r / 12.92;
    g = g > 0.04045 ? Math.pow((g + 0.055) / 1.055, 2.4) : g / 12.92;
    b = b > 0.04045 ? Math.pow((b + 0.055) / 1.055, 2.4) : b / 12.92;
    let x = (r * 0.4124564 + g * 0.3575761 + b * 0.1804375) / 0.95047;
    let y = (r * 0.2126729 + g * 0.7151522 + b * 0.0721750);
    let z = (r * 0.0193339 + g * 0.1191920 + b * 0.9503041) / 1.08883;
    const f = v => v > 0.008856 ? Math.pow(v, 1/3) : 7.787 * v + 16/116;
    return { L: 116 * f(y) - 16, a: 500 * (f(x) - f(y)), b: 200 * (f(y) - f(z)) };
  }

  // Delta-E (CIE76) distance
  function deltaE(lab1, lab2) {
    return Math.sqrt(
      Math.pow(lab1.L - lab2.L, 2) +
      Math.pow(lab1.a - lab2.a, 2) +
      Math.pow(lab1.b - lab2.b, 2)
    );
  }

  const els = document.querySelectorAll('*');
  const rawColors = new Set();
  const colorTuples = [];

  els.forEach(el => {
    const style = getComputedStyle(el);
    if (el.innerText && el.innerText.trim().length > 0) {
      rawColors.add(style.color);
    }
    if (style.backgroundColor !== 'rgba(0, 0, 0, 0)' && style.backgroundColor !== 'transparent') {
      rawColors.add(style.backgroundColor);
    }
  });

  // Parse all colors to tuples
  rawColors.forEach(c => {
    const parsed = parseColor(c);
    if (parsed) colorTuples.push({ str: c, rgb: parsed, lab: srgbToLab(parsed) });
  });

  // Cluster with Delta-E JND threshold of 3.0
  const JND = 3.0;
  const clustered = [];
  const assigned = new Array(colorTuples.length).fill(false);
  for (let i = 0; i < colorTuples.length; i++) {
    if (assigned[i]) continue;
    assigned[i] = true;
    const cluster = [colorTuples[i].str];
    for (let j = i + 1; j < colorTuples.length; j++) {
      if (assigned[j]) continue;
      if (deltaE(colorTuples[i].lab, colorTuples[j].lab) < JND) {
        assigned[j] = true;
        cluster.push(colorTuples[j].str);
      }
    }
    clustered.push(cluster);
  }

  const rawCount = rawColors.size;
  const clusteredCount = clustered.length;
  const grade = clusteredCount <= 15 ? 'good' : clusteredCount <= 25 ? 'warning' : 'bad';
  return { check: 'color_audit', rawDistinctColors: rawCount, clusteredDistinctColors: clusteredCount, grade, values: [...rawColors] };
})()
```

---

## Scoring Framework

### Tiered Sub-Scores

Report separate scores for each measurement tier:

| Tier | Description | Confidence |
|------|-------------|-----------|
| Deterministic [D] | Programmatic, reproducible | High |
| Heuristic [H] | Programmatic with <5% error | Medium |
| LLM-Assisted [J] | Pre-filtered LLM judgment | Lower |

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

If ANY check tagged as CRITICAL-severity fails, the total weighted score is capped at 50%.

### Compound Conditions

These checks require ALL conditions to pass (prevents gaming):

1. **Autofill (Cat 7)**: `autocomplete` present AND valid HTML token (not "on"/"off"/empty)
2. **Touch target (Cat 4)**: bounding rect >= 44x44 AND visible content area >= 30x30
3. **Pagination (Cat 9)**: Pagination elements present AND > 1 navigable page link
4. **Form label (Cat 4)**: `<label>` associated AND visible (rect area > 0) AND positioned near field
5. **Hover state (Cat 2)**: Style changes on :hover AND perceivable change in color/bg/border/shadow/transform/opacity
6. **Skeleton screen (Cat 8)**: Class found AND visible dimensions AND CSS animation present
7. **Heading frequency (Cat 4)**: Headings every 200-300 words AND heading texts are not all identical

### Conflicting Checks

These pairs can conflict. Report as linked findings, not independent:

1. Whitespace ratio (Cat 10) vs scroll depth (Cat 9)
2. Touch target size (Cat 4) vs nav item count (Cat 6) on mobile
3. CTA count = 1 (Cat 6) vs empty state CTA (Cat 9)
4. Information density (Cat 4) vs heading frequency (Cat 4)
5. Font sizes <= 6 (Cat 10) vs responsive type scales

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

## Scorecard Rules

### Scoring

- **`[D]` checks**: Pass (1) or Fail (0), with graduated scoring for the 7 checks listed above
- **`[H]` checks**: Pass (1) or Fail (0)
- **`[J]` checks**: Pass (1) or Fail (0), noted as LLM-assessed
- **warning** threshold = Pass (1) but noted as MINOR finding
- **bad** threshold = Fail (0)

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

```
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
| Component States | 1x | PASS | 8/8 | -- |
| Copy & Microcopy | 1x | PASS | 7/7 | -- |
| Accessibility | 2x | MINOR | 11/13 | 2 findings |
| Layout & Responsiveness | 1x | PASS | 6/6 | -- |
| Navigation & Wayfinding | 1x | MAJOR | 7/11 | 4 findings |
| Forms & Input | 1.5x | MINOR | 11/13 | 2 findings |
| Feedback & Response | 1x | PASS | 12/12 | -- |
| Data Display & Scalability | 1x | CRITICAL | 3/10 | 7 findings |
| Visual Complexity & Consistency | 0.5x | MINOR | 10/12 | 2 findings |

### Findings Detail
1. [CRITICAL] `[D]` **Unpaginated list with 87 items on /admin** — ...
2. [MAJOR] `[D]` **Nav has 12 top-level items** — ...
3. [MINOR] `[H]` **Alignment has 9 clusters** — ...
```
