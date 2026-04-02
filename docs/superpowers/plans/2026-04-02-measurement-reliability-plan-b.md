# Measurement Reliability: New Scripts + Missing Metrics (Plan B)

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add 25 new measurement scripts (20 heuristic + 5 pre-filters) for checks that previously had no programmatic measurement, add 6 missing metric categories (WCAG 2.2, dark mode, security headers, multi-run, font loading, resource hints), and add Shadow DOM traversal to all DOM queries.

**Architecture:** New measurement scripts are added to the existing `browser_evaluate` sections of each reference file. Missing metrics add new checks to existing categories or new sections. Shadow DOM traversal is a utility function prepended to scripts that do DOM queries.

**Tech Stack:** Markdown reference files with embedded JavaScript `browser_evaluate` snippets. No new dependencies.

**Spec:** `docs/superpowers/specs/2026-04-02-measurement-reliability-improvements.md` (Priorities 3 + 4)

**Depends on:** Plan A (structural + script fixes) must be completed first.

---

## File Map

| File | Changes |
|------|---------|
| `references/ux-auditor.md` | Add 15 new [H] measurement scripts for formerly-[J] checks, add 5 pre-filter scripts, add WCAG 2.2 checks to Category 4, add dark mode contrast note, add Shadow DOM utility |
| `references/mobile-ux-auditor.md` | Add 5 new [H] measurement scripts (iOS native feel, gesture pre-filters), add Shadow DOM utility |
| `references/performance-profiler.md` | Add font loading strategy checks (2 new static checks), add resource hint checks (2 new static checks), add multi-run measurement guidance, add SPA soft navigation documentation |
| `references/adversarial-breaker.md` | Add security headers check category (7 header checks) |
| `agents/adversarial-breaker.md` | Update attack categories list to include security headers |

---

## Task 1: Add Shadow DOM traversal utility to all reference files

A cross-cutting utility that all DOM-querying scripts need. Add it once per file, then reference it from scripts that do DOM queries.

**Files:**
- Modify: `references/ux-auditor.md`
- Modify: `references/mobile-ux-auditor.md`
- Modify: `references/performance-profiler.md`

- [ ] **Step 1: Add utility section to `references/ux-auditor.md`**

After the "Measurement Tier Legend" section and before Category 1, add:

```markdown
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

Use `deepQuerySelectorAll` instead of `document.querySelectorAll` in scripts that measure touch targets, form attributes, text elements, interactive elements, or any check where Shadow DOM elements would be missed. For performance, only use it when `document.querySelectorAll('[*-]').length > 0` suggests custom elements are present (the hyphen in tag names is required for custom elements).

**Known limitation:** Cross-origin iframes cannot be traversed. Third-party widgets (chat, payments, auth) inside cross-origin iframes are invisible to all measurement scripts. Document this in findings as "iframe content not audited."
```

- [ ] **Step 2: Add the same utility section to `references/mobile-ux-auditor.md`**

Same content, placed after the Measurement Tier Legend section.

- [ ] **Step 3: Add the same utility section to `references/performance-profiler.md`**

Same content, placed after the Measurement Tiers section. Add note: "Use `deepQuerySelectorAll` in the DOM Health script (Step 8) to count nodes inside shadow roots."

- [ ] **Step 4: Commit**

```bash
git add references/ux-auditor.md references/mobile-ux-auditor.md references/performance-profiler.md
git commit -m "feat: add Shadow DOM traversal utility to all reference files"
```

---

## Task 2: Add 15 new heuristic measurement scripts to UX-auditor

These scripts provide programmatic measurement for 15 checks that were previously `[J]` (LLM-judgment only). After adding these scripts, update the check tier tags from `[J]` to `[H]`.

**Files:**
- Modify: `references/ux-auditor.md`

- [ ] **Step 1: Read the current file**

Read `references/ux-auditor.md` in full to find the Measurement Scripts section.

- [ ] **Step 2: Add 15 new scripts after the existing scripts section**

Add a new subsection: `### Heuristic Measurement Scripts (Formerly LLM-Judgment)`

Add these 15 scripts, each as a named subsection with a complete `browser_evaluate` JavaScript snippet. The full implementations were provided by the conversion agents — here are the script names and key approaches:

1. **Icon Style Consistency** — SVG geometry: check `stroke-width` + `fill` attributes across icon-sized SVGs (12-64px). Classify as outline (stroke, no fill), filled (fill, no stroke), or ambiguous. Flag mixed styles.

2. **Error Message Specificity** — Dictionary of known-bad patterns: "something went wrong", "error", "oops", "try again", "invalid", etc. Flag messages matching these patterns or shorter than 4 words.

3. **Button Label Quality** — Dictionary of non-actionable labels ("submit", "ok", "yes", "go", "click here"). Dictionary of action verbs ("save", "create", "delete", "send", etc.). Flag buttons with generic single-word labels.

4. **Placeholder Text Quality** — Compare placeholder text to label text and field name. Flag: placeholder matches label (redundant), no label exists and placeholder used as label (anti-pattern), placeholder doesn't look like an example (no @, no format pattern, no "e.g.").

5. **Confirmation Message Quality** — Dictionary of generic success patterns ("success", "done", "ok", "completed"). Check if message mentions the subject ("profile", "account", "settings"). Flag single-word or generic messages.

6. **Empty State Quality (3-component)** — Structural check: container has (a) text with >= 5 words, (b) CTA button/link, (c) visual element (SVG/img > 20x20px). Score 0-3.

7. **Content Not Color-Only** — Check inline links for underline (flag if no `text-decoration: underline` and no bold/icon differentiation). Check form validation borders for accompanying text/icon. Check status indicators for text or aria-label.

8. **Dead End Detection** — Count visible outbound links, buttons, and navigation elements. Flag pages with zero outbound links and no nav.

9. **Error Message Actionability** — Check if error text contains a field name keyword AND a fix-guidance keyword ("must", "should", "at least", "format"). Measure pixel distance between error and input field.

10. **Default State Ambiguity Pre-filter** — Flag toggles without `aria-checked`, radio groups with no default selection, selects with placeholder first option, buttons with same background as parent.

11. **Tooltip Coverage Pre-filter** — Flag icon-only buttons (SVG/img without adjacent text or aria-label and no `title`/`data-tooltip`). Flag truncated text (`scrollWidth > clientWidth` with `overflow: hidden`) without `title`.

12. **Z-index Stacking Pre-filter** — Detect z-index values > 9999, overlapping elements where overlay-classed elements have lower z-index than content, and extreme z-index count (> 15 distinct values).

13. **Active/Pressed State Feedback** — Check CSS stylesheets for `:active` rules targeting interactive elements. Check for `transition` properties on buttons/links. Report coverage percentage.

14. **Error Recovery Path** — When error containers are visible, check for retry buttons, navigation links, editable form fields, or back-navigation links inside or near the error.

15. **Success Confirmation Infrastructure** — Detect toast/snackbar containers, `[role="status"]`, `[aria-live="polite"]`, success-classed elements. Check if forms have adjacent success containers.

- [ ] **Step 3: Update tier tags for the 15 checks**

Find each of the 15 checks in the category sections and change `[J]` to `[H]` for checks 1-9, 13-15. Keep checks 10-12 as `[J]` but add note: "Pre-filter script available — see Heuristic Measurement Scripts section."

Specifically:
- Cat 1: Icons `[J]` → `[H]` (script 1)
- Cat 2: Default state stays `[J]` (pre-filter script 10), Active `[J]` → `[H]` (script 13), Empty state stays `[J]` (but has script 6 as pre-filter), Error state `[J]` → `[H]` (scripts 2, 9)
- Cat 3: Error messages `[H]` (already, script 2 reinforces), Button labels `[H]` (already, script 3 reinforces), Placeholder `[H]` (already, script 4 reinforces), Confirmation `[H]` (already, script 5 reinforces), Empty states `[H]` (already, script 6 reinforces), Tooltips stays `[J]` (pre-filter script 11)
- Cat 4: Screen reader / color-only `[H]` → stays `[H]` (script 7 reinforces)
- Cat 5: Whitespace stays `[J]` (whitespace ratio script covers global; per-section is judgment), Z-index stays `[J]` (pre-filter script 12)
- Cat 6: Dead ends `[H]` (script 8 reinforces)
- Cat 7: Error actionability `[J]` → `[H]` (script 9), Undo stays `[J]`
- Cat 8: Action feedback `[J]` → `[H]` (script 15), Error recovery `[H]` (script 14 reinforces), Success `[H]` (script 15 reinforces), Optimistic stays `[J]` (pre-filter only)

- [ ] **Step 4: Update Per-Category Check Counts**

Update the count table to reflect the new tier distribution. After changes:
- `[D]`: ~48 checks (unchanged from Plan A)
- `[H]`: ~41 checks (up from ~37, as ~4 moved from [J] to [H])
- `[J]`: ~10 checks (down from ~14, with pre-filter scripts available)

- [ ] **Step 5: Commit**

```bash
git add references/ux-auditor.md
git commit -m "feat: add 15 heuristic measurement scripts for formerly-LLM-judgment checks"
```

---

## Task 3: Add 5 new heuristic scripts to mobile UX-auditor

Add measurement scripts for mobile-specific checks that were `[J]`.

**Files:**
- Modify: `references/mobile-ux-auditor.md`

- [ ] **Step 1: Read the current file**

Read `references/mobile-ux-auditor.md` in full.

- [ ] **Step 2: Add 5 new scripts**

Add after the existing measurement scripts section:

1. **iOS Native Feel Composite** — Detect MDC/MUI classes, unstyled checkboxes (< 30px, no toggle class), FAB geometry (fixed + circular + small), hamburger patterns (class-based + SVG), breadcrumbs on mobile, Android-style toasts (fixed bottom).

2. **Gesture Support Pre-filter** — Detect gesture libraries, `touch-action: none` blockers on large elements, swipe-related classes, overscroll-behavior CSS, scrollable list containers missing PTR.

3. **Entrance/Exit Asymmetry Pre-filter** — Cannot detect programmatically from static CSS (transition-duration is single value). Flag as `{ available: false, reason: 'requires interaction testing' }`. Note: keep as `[J]`.

4. **Material Design Styling Detection** — Check for `mdc-`, `mat-`, `Mui`, `ripple` class prefixes. Count occurrences. Flag if > 0.

5. **Component Pattern Detection (Checkbox vs Toggle)** — Find `input[type="checkbox"]` elements. Classify as "web checkbox" (< 30px, no toggle/switch class on element or parent) vs "styled toggle" (has toggle/switch class or dimensions > 30px). Flag web checkboxes.

- [ ] **Step 3: Update tier tags**

- Cat 3: Material Design `[J]` → `[H]` (script 4), Component patterns `[J]` → `[H]` (script 5)
- Cat 9: Gesture support stays `[J]` (pre-filter script 2 available)
- Cat 10: Entrance/exit asymmetry stays `[J]` (script 3 returns unavailable)

- [ ] **Step 4: Commit**

```bash
git add references/mobile-ux-auditor.md
git commit -m "feat: add 5 heuristic measurement scripts for mobile checks"
```

---

## Task 4: Add WCAG 2.2 AA checks to UX-auditor

Add 6 new accessibility checks from WCAG 2.2 to Category 4 of the UX-auditor.

**Files:**
- Modify: `references/ux-auditor.md`

- [ ] **Step 1: Add 6 new checks to Category 4 (Accessibility)**

After the existing Cognitive Load Checks, add a new subsection:

```markdown
### WCAG 2.2 Checks

- [ ] `[H]` Focus Not Obscured (WCAG 2.4.11): When an element receives focus, it must not be entirely hidden behind sticky headers, footers, or other fixed-position elements `[research: WCAG 2.2]`
- [ ] `[D]` Target Size — Desktop (WCAG 2.5.8): Interactive elements must be at least 24x24 CSS px, or have sufficient spacing (24px offset) `[research: WCAG 2.2]`
- [ ] `[H]` Dragging Movements (WCAG 2.5.7): All drag operations must have single-pointer alternatives (button, click, or keyboard) `[research: WCAG 2.2]`
- [ ] `[H]` Consistent Help (WCAG 3.2.6): Help mechanisms (contact info, FAQ link, chat) must appear in the same relative position across pages `[research: WCAG 2.2]`
- [ ] `[H]` Redundant Entry (WCAG 3.3.7): Information previously entered by the user must be auto-populated or available for selection `[research: WCAG 2.2]`
- [ ] `[H]` Accessible Authentication (WCAG 3.3.8): Authentication must not require cognitive function tests (CAPTCHA puzzles) unless an alternative is provided `[research: WCAG 2.2]`
```

- [ ] **Step 2: Add measurement scripts for the 6 new checks**

Add to the Measurement Scripts section:

1. **Target Size Desktop (24x24)** — Same approach as touch target check but with 24px threshold and 24px spacing offset alternative. `[D]`

2. **Focus Not Obscured** — Tab through focusable elements (requires Playwright). For each, check if bounding rect overlaps with any `position: fixed/sticky` element. `[H]` — requires interaction.

3. **Dragging Movements** — Detect `[draggable="true"]` elements and elements with drag-related event attributes (`ondragstart`). Flag those without a visible button/link alternative nearby. `[H]`

4. **Consistent Help** — Detect help-related elements (links with "help", "support", "contact", "FAQ" text; chat widgets). Record their position. Cross-screen comparison needed (agent compares across pages). `[H]`

5. **Redundant Entry** — Detect multi-step forms. Check if fields in later steps that match earlier field names/types are pre-populated. `[H]` — requires interaction.

6. **Accessible Authentication** — Detect CAPTCHA elements (`iframe[src*="captcha"]`, `[class*="captcha"]`, `[class*="recaptcha"]`). Flag if no alternative is provided. `[H]`

- [ ] **Step 3: Update category check count**

Category 4 goes from 13 to 19 checks.

- [ ] **Step 4: Commit**

```bash
git add references/ux-auditor.md
git commit -m "feat: add 6 WCAG 2.2 AA accessibility checks"
```

---

## Task 5: Add dark mode contrast testing note

Add guidance for running contrast checks in both light and dark mode.

**Files:**
- Modify: `references/ux-auditor.md`
- Modify: `references/mobile-ux-auditor.md`

- [ ] **Step 1: Add dark mode note to UX-auditor**

In the contrast ratio check description (Category 4), add:

```markdown
**Dark Mode Testing:** Run contrast checks twice — once in default mode and once after `page.emulateMedia({ colorScheme: 'dark' })`. Common failures: gray text on dark backgrounds, muted brand colors, focus rings invisible on dark backgrounds. Report both mode results separately.
```

- [ ] **Step 2: Add same note to mobile UX-auditor**

In Category 5 (Mobile Typography), under the contrast checks, add the same dark mode note.

- [ ] **Step 3: Commit**

```bash
git add references/ux-auditor.md references/mobile-ux-auditor.md
git commit -m "feat: add dark mode contrast testing guidance"
```

---

## Task 6: Add security headers to adversarial-breaker

Add a new "Security Headers" category to the adversarial agent's attack categories.

**Files:**
- Modify: `references/adversarial-breaker.md`
- Modify: `agents/adversarial-breaker.md`

- [ ] **Step 1: Read both files**

Read `references/adversarial-breaker.md` and `agents/adversarial-breaker.md`.

- [ ] **Step 2: Add Security Headers category to reference file**

After the existing 6 attack categories, add:

```markdown
### 7. Security Headers

Check HTTP response headers for security best practices. Use `browser_evaluate` to read headers via `fetch()` to the current page:

```javascript
(() => {
  return fetch(window.location.href, { method: 'HEAD' })
    .then(r => {
      const headers = {};
      r.headers.forEach((v, k) => headers[k] = v);
      
      const checks = {
        csp: {
          present: !!headers['content-security-policy'],
          value: (headers['content-security-policy'] || '').slice(0, 200),
          severity: 'HIGH'
        },
        hsts: {
          present: !!headers['strict-transport-security'],
          value: headers['strict-transport-security'] || '',
          hasMaxAge: (headers['strict-transport-security'] || '').includes('max-age'),
          severity: 'HIGH'
        },
        xFrameOptions: {
          present: !!headers['x-frame-options'],
          value: headers['x-frame-options'] || '',
          severity: 'MEDIUM'
        },
        xContentType: {
          present: !!headers['x-content-type-options'],
          isNosniff: headers['x-content-type-options'] === 'nosniff',
          severity: 'MEDIUM'
        },
        referrerPolicy: {
          present: !!headers['referrer-policy'],
          value: headers['referrer-policy'] || '',
          severity: 'LOW'
        },
        permissionsPolicy: {
          present: !!headers['permissions-policy'],
          value: (headers['permissions-policy'] || '').slice(0, 200),
          severity: 'MEDIUM'
        }
      };
      
      // Check cookies for security flags
      const cookies = document.cookie.split(';').map(c => c.trim());
      // Note: HttpOnly cookies are not visible to JS — their absence here is expected
      
      // Check for SRI on third-party scripts
      const scripts = document.querySelectorAll('script[src]');
      const thirdParty = [...scripts].filter(s => {
        try { return new URL(s.src).origin !== window.location.origin; } 
        catch { return false; }
      });
      const withIntegrity = thirdParty.filter(s => s.hasAttribute('integrity'));
      
      checks.sri = {
        thirdPartyScripts: thirdParty.length,
        withIntegrity: withIntegrity.length,
        missingIntegrity: thirdParty.length - withIntegrity.length,
        severity: thirdParty.length > 0 && withIntegrity.length === 0 ? 'MEDIUM' : 'LOW'
      };
      
      const missing = Object.entries(checks).filter(([k, v]) => !v.present && v.severity !== undefined);
      
      return {
        check: 'security_headers',
        headers: checks,
        missingCount: missing.length,
        missingSeverity: missing.map(([k, v]) => ({ header: k, severity: v.severity }))
      };
    })
    .catch(e => ({ check: 'security_headers', available: false, reason: e.message }));
})()
```

- Content-Security-Policy: prevents XSS, data injection. **HIGH** if missing.
- Strict-Transport-Security: forces HTTPS. **HIGH** if missing on HTTPS site.
- X-Frame-Options: prevents clickjacking. **MEDIUM** if missing.
- X-Content-Type-Options: nosniff prevents MIME sniffing. **MEDIUM** if missing.
- Referrer-Policy: controls information leakage. **LOW** if missing.
- Permissions-Policy: restricts browser features. **MEDIUM** if missing.
- Subresource Integrity: protects third-party scripts. **MEDIUM** if third-party scripts lack `integrity`.
```

- [ ] **Step 3: Update adversarial agent's attack category list**

In `agents/adversarial-breaker.md`, find the "Attack Categories:" list and add:

```
7. Security Headers
```

- [ ] **Step 4: Commit**

```bash
git add references/adversarial-breaker.md agents/adversarial-breaker.md
git commit -m "feat: add security headers check category to adversarial agent"
```

---

## Task 7: Add performance profiler missing checks

Add font loading strategy checks, resource hint checks, multi-run guidance, and SPA documentation.

**Files:**
- Modify: `references/performance-profiler.md`

- [ ] **Step 1: Read the current file**

Read `references/performance-profiler.md` in full.

- [ ] **Step 2: Add font loading strategy checks to static analysis**

After the existing static analysis Category 4 (Images & Assets), or as new items in the appropriate category, add:

```markdown
#### Font Loading Strategy

| # | Check | What to look for | Severity | Vitals |
|---|-------|-----------------|----------|--------|
| FL1 | Missing `font-display` on @font-face | @font-face rules without `font-display: swap` or `font-display: optional`. Causes invisible text during font load (FOIT). | HIGH | LCP, CLS |
| FL2 | Web fonts not preloaded | Critical web fonts (used in above-fold text) not loaded via `<link rel="preload" as="font">`. Delays text rendering. | MEDIUM | LCP |
| FL3 | Missing `size-adjust` on fallback fonts | Font fallback declarations without `size-adjust`, `ascent-override`, or `descent-override`. Causes layout shift when web font loads. | LOW | CLS |
```

- [ ] **Step 3: Add resource hint checks**

Add to the static analysis:

```markdown
#### Resource Hints

| # | Check | What to look for | Severity | Vitals |
|---|-------|-----------------|----------|--------|
| RH1 | Missing `preconnect` for critical third-party origins | Third-party scripts (analytics, CDN, API) loaded without `<link rel="preconnect">`. Saves DNS+TCP+TLS time. | MEDIUM | TTFB, LCP |
| RH2 | Missing `fetchpriority="high"` on LCP image | The LCP image element (usually hero/banner) should have `fetchpriority="high"` to prioritize its loading. | HIGH | LCP |
| RH3 | Missing `dns-prefetch` for external origins | External origins used later in the page (lazy-loaded content, deferred scripts) without `<link rel="dns-prefetch">`. | LOW | TTFB |
| RH4 | Excessive `preload` usage | More than 3-4 `<link rel="preload">` resources. Overuse negates the priority benefit and wastes bandwidth. | LOW | LCP |
```

- [ ] **Step 4: Add multi-run measurement guidance**

Add a new section after the Per-Route Profiling Loop:

```markdown
## Multiple-Run Measurement

For reliable before/after comparison, timing-based metrics require multiple runs.

### Protocol

1. Run the full profiling loop **3 times** for each route
2. For each metric, report the **median** of the 3 runs
3. For before/after comparisons:
   - Run 3 times before changes, 3 times after
   - Compare median values
   - Flag changes < 10% as "within measurement noise"
   - Only report improvements/regressions where the P75 of "after" consistently differs from the P75 of "before"

### Metric Stability

| Metric | Stability | Recommended Runs |
|--------|-----------|-----------------|
| TTFB | Low (network-dependent) | 5 runs, report median |
| FCP | Medium | 3 runs |
| LCP | Medium (cache-dependent) | 3 runs |
| CLS | Low-Medium (timing-dependent) | 3 runs |
| TBT | Medium (CPU-dependent) | 3 runs |
| DOM nodes | High (deterministic) | 1 run sufficient |
| Bundle sizes | High (deterministic) | 1 run sufficient |
| Resource count | High (deterministic) | 1 run sufficient |

### Environment Consistency

For valid comparison, hold constant:
- Viewport dimensions (already specified)
- Browser (Chromium recommended)
- Network conditions (unthrottled, or consistent throttling)
- Cache state (clear cache between runs)
- Data state (same database content)
```

- [ ] **Step 5: Add SPA soft navigation documentation**

Add a note in the Per-Route Profiling Loop section:

```markdown
### SPA Soft Navigation Limitation

The profiling loop uses `browser_navigate` for each route, which triggers full page loads. In SPAs with client-side routing (Next.js App Router, React Router), real users navigate via link clicks that produce soft navigations. Metrics from hard navigations may differ significantly from the in-app experience.

**Mitigation:** For SPA routes, consider an additional measurement pass that:
1. Navigate to the app's entry point
2. Click through to the target route via internal links
3. Measure metrics after the soft navigation

This requires Playwright interaction (click, wait) rather than `browser_navigate`, and is not yet automated in the profiling loop. Document this gap in findings when auditing SPAs.
```

- [ ] **Step 6: Commit**

```bash
git add references/performance-profiler.md
git commit -m "feat: add font loading, resource hints, multi-run guidance, and SPA docs to performance profiler"
```

---

## Task 8: Run validation

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
git commit -m "fix: address validation issues from Plan B changes"
```

---

## Task Summary

| Task | Description | Dependencies | Files |
|------|-------------|--------------|-------|
| 1 | Shadow DOM traversal utility | None | 3 reference files |
| 2 | 15 new UX heuristic scripts + tier updates | Task 1 | ux-auditor.md |
| 3 | 5 new mobile heuristic scripts + tier updates | Task 1 | mobile-ux-auditor.md |
| 4 | 6 WCAG 2.2 AA checks | Task 2 | ux-auditor.md |
| 5 | Dark mode contrast testing guidance | Tasks 2, 3 | both ux-auditor files |
| 6 | Security headers for adversarial agent | None | adversarial ref + agent |
| 7 | Performance profiler missing checks + docs | None | performance-profiler.md |
| 8 | Validation | All | — |

**Parallelizable groups:**
- Group A: Task 1 (Shadow DOM — touches all 3 files, do first)
- Group B: Tasks 2 + 4 + 5 (UX-auditor additions — sequential, same file)
- Group C: Task 3 (mobile — independent file)
- Group D: Task 6 (adversarial — independent files)
- Group E: Task 7 (performance — independent file)
- Group F: Task 8 (validation — depends on all)

After Task 1, Groups B/C/D/E can run in parallel.
