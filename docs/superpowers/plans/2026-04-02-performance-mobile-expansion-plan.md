# Performance & Mobile Expansion Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Expand qa-skills from 3 personas to 5 (adding performance profiler + mobile UX auditor), expand the UX-auditor to 10 categories with quantifiable metrics and binary scorecards, restructure all agents to use dedicated reference files, and wire everything into `/run-qa`.

**Architecture:** Each persona has an agent markdown file (process + output format) and a dedicated reference file (detailed checks + measurement scripts). The `/run-qa` command dispatches any combination of the 5 personas. All UX checks produce numeric values for before/after comparison via binary scorecards.

**Tech Stack:** Markdown agent/reference files, Playwright MCP `browser_evaluate` for runtime measurements, bash scripts for build analysis, YAML frontmatter for agent metadata.

**Spec:** `docs/superpowers/specs/2026-04-02-performance-mobile-expansion-design.md`

---

## Task 1: Delete the performance-profiler skill

Remove the skill that the new agent replaces. Do this first to avoid confusion during development.

**Files:**
- Delete: `skills/performance-profiler/SKILL.md`
- Delete: `skills/performance-profiler/` (directory)
- Delete: `scripts/performance/compare-metrics.sh`

- [ ] **Step 1: Delete the skill directory and compare script**

```bash
rm -rf skills/performance-profiler
rm scripts/performance/compare-metrics.sh
```

- [ ] **Step 2: Verify deletion**

```bash
ls skills/performance-profiler 2>&1  # Should say "No such file or directory"
ls scripts/performance/compare-metrics.sh 2>&1  # Should say "No such file or directory"
ls scripts/performance/parse-build-output.sh  # Should still exist
ls scripts/performance/run-lighthouse.sh  # Should still exist
```

- [ ] **Step 3: Commit**

```bash
git add -u skills/performance-profiler scripts/performance/compare-metrics.sh
git commit -m "chore: remove performance-profiler skill (replaced by agent)"
```

---

## Task 2: Create the performance profiler reference file

Consolidate the two existing reference files into one, add the new metrics (FCP, TBT, DOM health, Long Tasks, memory), and add expanded thresholds.

**Files:**
- Read: `references/performance-checks.md` (existing — 73 static checks)
- Read: `references/web-vitals-measurement.md` (existing — runtime collection scripts)
- Create: `references/performance-profiler.md` (consolidated)
- Delete: `references/performance-checks.md`
- Delete: `references/web-vitals-measurement.md`

- [ ] **Step 1: Read existing reference files**

Read `references/performance-checks.md` and `references/web-vitals-measurement.md` in full.

- [ ] **Step 2: Create `references/performance-profiler.md`**

Write a new file that contains:

1. **Rating Thresholds** — the expanded table from the spec (LCP, CLS, INP, FCP, TTFB, TBT, Total JS, Total page weight, DOM nodes, Long Tasks, HTTP requests, JS Execution Time, Lighthouse score) with Good/Warning/Bad columns
2. **Runtime Measurement Scripts** — all `browser_evaluate` JavaScript snippets:
   - Navigation Timing (TTFB, DOM content loaded, full load, transfer size) — from existing `web-vitals-measurement.md`
   - LCP collection — from existing
   - CLS collection — from existing
   - Resource Loading (heavy resources, JS/CSS/image breakdown) — from existing
   - **NEW: FCP collection** via `PerformanceObserver('paint')`:
     ```javascript
     () => {
       const entries = performance.getEntriesByType('paint');
       const fcp = entries.find(e => e.name === 'first-contentful-paint');
       return fcp ? { available: true, fcp_ms: Math.round(fcp.startTime) } : { available: false };
     }
     ```
   - **NEW: Long Tasks collection** via `PerformanceObserver('longtask')`:
     ```javascript
     () => {
       return new Promise(resolve => {
         const tasks = [];
         const observer = new PerformanceObserver(list => {
           for (const entry of list.getEntries()) {
             tasks.push({ duration_ms: Math.round(entry.duration), startTime: Math.round(entry.startTime) });
           }
         });
         observer.observe({ type: 'longtask', buffered: true });
         setTimeout(() => {
           observer.disconnect();
           const tbt = tasks.reduce((sum, t) => sum + Math.max(0, t.duration_ms - 50), 0);
           resolve({
             count: tasks.length,
             tbt_ms: tbt,
             longest_ms: tasks.length > 0 ? Math.max(...tasks.map(t => t.duration_ms)) : 0,
             tasks: tasks.slice(0, 10)
           });
         }, 5000);
       });
     }
     ```
   - **NEW: DOM Health** measurement:
     ```javascript
     () => {
       const all = document.querySelectorAll('*');
       let maxDepth = 0;
       let maxChildren = 0;
       all.forEach(el => {
         let depth = 0;
         let node = el;
         while (node.parentElement) { depth++; node = node.parentElement; }
         if (depth > maxDepth) maxDepth = depth;
         if (el.children.length > maxChildren) maxChildren = el.children.length;
       });
       return { total_nodes: all.length, max_depth: maxDepth, max_children: maxChildren };
     }
     ```
   - **NEW: Memory snapshot** (Chromium only):
     ```javascript
     () => {
       if (performance.memory) {
         return {
           available: true,
           used_mb: Math.round(performance.memory.usedJSHeapSize / 1048576 * 100) / 100,
           total_mb: Math.round(performance.memory.totalJSHeapSize / 1048576 * 100) / 100,
           limit_mb: Math.round(performance.memory.jsHeapSizeLimit / 1048576 * 100) / 100
         };
       }
       return { available: false };
     }
     ```
3. **Viewport Profiling** — desktop (1280x720) and mobile (393x852) instructions from existing
4. **Per-Route Profiling Loop** — from existing
5. **Static Analysis Checks** — all 73 checks across 7 categories, copied verbatim from existing `performance-checks.md`

- [ ] **Step 3: Delete old reference files**

```bash
rm references/performance-checks.md references/web-vitals-measurement.md
```

- [ ] **Step 4: Verify**

```bash
ls references/performance-profiler.md  # Should exist
ls references/performance-checks.md 2>&1  # Should be gone
ls references/web-vitals-measurement.md 2>&1  # Should be gone
```

- [ ] **Step 5: Commit**

```bash
git add references/performance-profiler.md
git add -u references/performance-checks.md references/web-vitals-measurement.md
git commit -m "feat: create consolidated performance profiler reference file"
```

---

## Task 3: Create the performance profiler agent

**Files:**
- Create: `agents/performance-profiler.md`

- [ ] **Step 1: Create `agents/performance-profiler.md`**

Write the agent file with:
- YAML frontmatter: `name: performance-profiler`, `description:` matching the spec
- 3 examples (usage patterns)
- Role description: report-only performance measurement
- Execution process:
  1. Auth setup (same pattern as other agents — load storageState from spawn prompt)
  2. Runtime profiling: for each route, navigate + collect all metrics from `references/performance-profiler.md`
  3. Static analysis: read the 73 checks from `references/performance-profiler.md`, scan codebase
  4. Build analysis: run `scripts/performance/parse-build-output.sh` if `next build` output is available
  5. Report: per-route metrics table + binary scorecard + findings with severity
- Reference instruction: "Read `references/performance-profiler.md` for detailed measurement scripts, thresholds, and static analysis checks."
- Output format: the scorecard + table + findings from spec

- [ ] **Step 2: Validate frontmatter**

```bash
head -5 agents/performance-profiler.md  # Should show --- / name / description / ---
```

- [ ] **Step 3: Commit**

```bash
git add agents/performance-profiler.md
git commit -m "feat: add performance-profiler agent"
```

---

## Task 4: Create the UX-auditor reference file

Extract detail from the existing `agents/ux-auditor.md` inline content, add the 2 new categories, and add enhanced checks to 4 existing categories.

**Files:**
- Read: `agents/ux-auditor.md` (existing — contains inline rubric)
- Create: `references/ux-auditor.md`

- [ ] **Step 1: Read existing ux-auditor agent**

Read `agents/ux-auditor.md` in full to capture all inline rubric content.

- [ ] **Step 2: Create `references/ux-auditor.md`**

Write a reference file containing:

1. **All 10 categories** with every check, threshold, and measurement method:
   - Categories 1-8: existing checks from the agent (Visual Consistency, Component States, Copy & Microcopy, Accessibility, Layout & Responsiveness, Navigation & Wayfinding, Forms & Input, Feedback & Response) — each enhanced with the new checks from the spec
   - Category 9: Data Display & Scalability (NEW — 10 checks from spec)
   - Category 10: Visual Complexity & Consistency (NEW — 12 checks from spec)

2. **`browser_evaluate` measurement scripts** for automatable checks:
   - Scroll depth ratio: `document.documentElement.scrollHeight / document.documentElement.clientHeight`
   - Repeated item count: count elements sharing same class pattern
   - Font size/family/combo audit script
   - Spacing grid conformance script (check `% 4 === 0`)
   - Alignment clustering script
   - Visual balance (Ngo) formula implementation
   - Whitespace ratio calculation
   - Flesch-Kincaid grade level formula:
     ```
     0.39 * (total_words / total_sentences) + 11.8 * (total_syllables / total_words) - 15.59
     ```
   - Flesch Reading Ease formula:
     ```
     206.835 - 1.015 * (total_words / total_sentences) - 84.6 * (total_syllables / total_words)
     ```
   - Heading frequency: words between `h1`-`h6` elements
   - Information density: word count within viewport bounds
   - Nav item count, dropdown count, CTA count scripts

3. **Severity grading criteria**: PASS / MINOR / MAJOR / CRITICAL definitions per category

4. **Binary scorecard rules**: each check maps to Pass (1) or Fail (0), total is X/N per screen

- [ ] **Step 3: Commit**

```bash
git add references/ux-auditor.md
git commit -m "feat: create ux-auditor reference with 10 categories and measurement scripts"
```

---

## Task 5: Slim down the UX-auditor agent

Replace inline rubric detail with a reference pointer. Keep the agent focused on process and output format.

**Files:**
- Modify: `agents/ux-auditor.md`

- [ ] **Step 1: Read current agent**

Read `agents/ux-auditor.md` in full.

- [ ] **Step 2: Rewrite `agents/ux-auditor.md`**

Keep:
- YAML frontmatter (name, description)
- 3 examples
- Role description paragraph
- Core responsibilities list
- Execution process (auth setup, screen inspection, cross-screen consistency, report)
- Output format (updated to include binary scorecard and 10 categories)
- Principles section

Replace all inline rubric checklists (categories 1-8) with:
```
Read `references/ux-auditor.md` for the complete 10-category rubric with detailed checks, thresholds, measurement scripts, and grading criteria.
```

Add mention of the 2 new categories in the role description.

- [ ] **Step 3: Verify frontmatter still valid**

```bash
head -5 agents/ux-auditor.md
```

- [ ] **Step 4: Commit**

```bash
git add agents/ux-auditor.md
git commit -m "refactor: slim ux-auditor agent, move rubric detail to reference file"
```

---

## Task 6: Create the mobile UX auditor reference file

**Files:**
- Create: `references/mobile-ux-auditor.md`

- [ ] **Step 1: Create `references/mobile-ux-auditor.md`**

Write a reference file containing all 10 mobile categories with every check, threshold, measurement method, and `browser_evaluate` scripts:

1. **Touch & Interaction** (7 checks) — tap target measurement script, thumb zone calculation, input height check, label visibility
2. **iOS Safari Specific** (5 checks) — 100vh detection, input zoom check, safe area validation, fixed bottom detection, viewport unit scan
3. **iOS Native Feel** (6 checks) — hamburger detection heuristics (`[class*="hamburger"]`, `[class*="burger"]`, `[aria-label*="menu"]` with hidden child navs), FAB detection (`position: fixed` circular buttons), component pattern detection
4. **Viewport & Responsive** (6 checks) — meta viewport parser, overflow check, orientation test procedure, viewport utilization calculation
5. **Mobile Typography** (10 checks) — font size audit script, line length calculation, text scaling test procedure at 200%, iOS type scale reference table (Large Title 34pt through Caption2 11pt)
6. **Mobile Form UX** (8 checks) — input type/inputmode/autocomplete cross-reference script, keyboard type matching logic, single-column detection
7. **Interstitials & Overlays** (4 checks) — overlay area computation script, sticky banner measurement, close button size check
8. **Mobile Accessibility** (6 checks) — touch target measurement, motion media query detection, focus obscured check procedure
9. **Gestures & Interaction** (5 checks) — pull-to-refresh detection, swipe-back test procedure, skeleton screen presence check
10. **Animation & Motion** (5 checks) — CSS transition/animation duration audit script, easing curve check, MD3 elevation levels reference (0/1/3/6/8/12dp), MD3 duration tokens reference (50ms-1000ms), MD3 easing curves reference

Include the **39-point binary scorecard template** from the mobile UX success criteria file (adapted to the 10-category structure with ~56 total checks).

- [ ] **Step 2: Commit**

```bash
git add references/mobile-ux-auditor.md
git commit -m "feat: create mobile-ux-auditor reference with 10 categories and measurement scripts"
```

---

## Task 7: Create the mobile UX auditor agent

**Files:**
- Create: `agents/mobile-ux-auditor.md`

- [ ] **Step 1: Create `agents/mobile-ux-auditor.md`**

Write the agent file with:
- YAML frontmatter: `name: mobile-ux-auditor`, `description:` matching the spec
- 3 examples
- Role description: comprehensive mobile UX audit at 393x852 viewport, report-only
- Execution process:
  1. Auth setup (same pattern as other agents)
  2. Set viewport: `browser_resize width=393 height=852`
  3. For each screen: navigate, apply all 10 categories from `references/mobile-ux-auditor.md`, take screenshots
  4. Cross-screen consistency check at mobile viewport
  5. Report: binary scorecard + graded rubric per screen + findings
- Reference instruction: "Read `references/mobile-ux-auditor.md` for the complete 10-category rubric with detailed checks, thresholds, and measurement scripts. Also read `references/ios-hig-requirements.md` and `references/ios-hig-anti-patterns.md` for iOS-specific standards."
- Output format: the scorecard from spec (X/56 total)
- Principles: same style as ux-auditor — specific, honest, prioritize by user impact

- [ ] **Step 2: Validate frontmatter**

```bash
head -5 agents/mobile-ux-auditor.md
```

- [ ] **Step 3: Commit**

```bash
git add agents/mobile-ux-auditor.md
git commit -m "feat: add mobile-ux-auditor agent with 10-category rubric"
```

---

## Task 8: Extract smoke-tester reference file and slim agent

**Files:**
- Read: `agents/smoke-tester.md` (existing)
- Create: `references/smoke-tester.md`
- Modify: `agents/smoke-tester.md`

- [ ] **Step 1: Read existing smoke-tester agent**

Read `agents/smoke-tester.md` in full.

- [ ] **Step 2: Create `references/smoke-tester.md`**

Extract from the agent into the reference file:
- Auth setup code (storageState loading JavaScript snippet)
- Action mapping table (workflow language -> Playwright MCP tool mapping)
- Quality standards (never skip a step, always snapshot, etc.)
- No-workflow mode (coverage gap) procedures and the 5-point smoke check
- Report format template (markdown table format)

- [ ] **Step 3: Slim down `agents/smoke-tester.md`**

Keep in the agent:
- YAML frontmatter
- 3 examples
- Role description paragraph
- Core responsibilities list (6 items)
- Execution process outline (4 phases: auth, parse, execute, report) — but replace inline detail with reference pointer

Replace inline detail with:
```
Read `references/smoke-tester.md` for auth setup code, action mapping table, quality standards, no-workflow mode procedures, and report format template.
```

- [ ] **Step 4: Verify frontmatter**

```bash
head -5 agents/smoke-tester.md
```

- [ ] **Step 5: Commit**

```bash
git add references/smoke-tester.md agents/smoke-tester.md
git commit -m "refactor: extract smoke-tester reference file, slim agent"
```

---

## Task 9: Extract adversarial-breaker reference file and slim agent

**Files:**
- Read: `agents/adversarial-breaker.md` (existing)
- Create: `references/adversarial-breaker.md`
- Modify: `agents/adversarial-breaker.md`

- [ ] **Step 1: Read existing adversarial-breaker agent**

Read `agents/adversarial-breaker.md` in full.

- [ ] **Step 2: Create `references/adversarial-breaker.md`**

Extract from the agent:
- Auth setup code (storageState loading + clearing + multi-profile switching)
- All 6 attack categories with full detail (Input Abuse, Sequence Breaking, Auth Boundary Testing, State Corruption, Error Handling, Client-Side Security)
- Severity rating definitions table
- Report format template
- Principles section

- [ ] **Step 3: Slim down `agents/adversarial-breaker.md`**

Keep in the agent:
- YAML frontmatter
- 3 examples
- Role description paragraph
- Core responsibilities list
- Execution process outline (4 phases: recon, auth, attack, report) — brief summary only

Replace inline detail with:
```
Read `references/adversarial-breaker.md` for auth setup code, all 6 attack category checklists, severity definitions, and report format template.
```

- [ ] **Step 4: Verify frontmatter**

```bash
head -5 agents/adversarial-breaker.md
```

- [ ] **Step 5: Commit**

```bash
git add references/adversarial-breaker.md agents/adversarial-breaker.md
git commit -m "refactor: extract adversarial-breaker reference file, slim agent"
```

---

## Task 10: Update `/run-qa` command

Add performance and mobile personas to argument parsing, dispatch logic, and report format.

**Files:**
- Modify: `commands/run-qa.md`

- [ ] **Step 1: Read current command**

Read `commands/run-qa.md` in full.

- [ ] **Step 2: Update Phase 1 (argument parsing)**

Change the agent selection table from:

```
| smoke | smoke-tester only |
| ux | ux-auditor only |
| adversarial | adversarial-breaker only |
| all | All three agents per screen |
```

To:

```
| smoke | smoke-tester only |
| ux | ux-auditor only |
| adversarial | adversarial-breaker only |
| performance | performance-profiler only |
| mobile | mobile-ux-auditor only |
| all | All five agents |
```

Also update `argument-hint` in YAML frontmatter to: `"[smoke|ux|adversarial|performance|mobile|all] [--url URL]"`

- [ ] **Step 3: Update Phase 3 (pre-flight — agent selection prompt)**

Change the "which agent(s)" prompt from listing 3 options + "all three" to listing 5 options + "all five":

```
1. smoke-tester — Quick pass/fail on each screen (fastest)
2. ux-auditor — Obsessive UX rubric on each screen (thorough, 10 categories)
3. adversarial-breaker — Try to break each flow (deepest)
4. performance-profiler — Measure Web Vitals, bundle size, code patterns (report-only)
5. mobile-ux-auditor — Mobile UX audit at 393x852 viewport (10 categories, iOS + web)
6. All five — Full QA suite
```

Also update the profile assignment prompt to include `performance-profiler` and `mobile-ux-auditor`.

- [ ] **Step 4: Update Phase 4 (dispatch — agent spawn templates)**

Add dispatch templates for the two new agents:

**Performance-profiler template** (dispatched per route or once for all routes):
```
You are operating as the performance-profiler QA agent.

Target routes: [list of routes from manifest]
Auth required: [yes/no]
Auth profile to use: [profile name]
Auth profile path: .playwright/profiles/[profile-name].json

[auth loading code — same pattern as other agents]

[AGENT SYSTEM PROMPT — insert body of agents/performance-profiler.md]

Base URL: [base_url]

Begin your audit now. Profile each route, run static analysis, and return
your findings in the output format specified in your system prompt.
```

**Mobile-ux-auditor template** (dispatched per screen):
```
You are operating as the mobile-ux-auditor QA agent.

Target: [screen name] at [base_url][example_url]
Auth required: [yes/no]
Auth profile to use: [profile name]
Auth profile path: .playwright/profiles/[profile-name].json

[auth loading code — same pattern as other agents]

IMPORTANT: Set mobile viewport before inspection:
  browser_resize width=393 height=852

[AGENT SYSTEM PROMPT — insert body of agents/mobile-ux-auditor.md]

Base URL: [base_url]
Screen URL: [full_url]

Begin your audit now. When complete, return your findings in the output
format specified in your system prompt.
```

- [ ] **Step 5: Update Phase 5 (report format)**

Update the coverage table to include all 5 personas:

```
| Screen | Smoke | UX (X/75) | Mobile (X/56) | Perf | Adversarial |
```

Update the report structure to include all 5 sections (only dispatched ones appear).

- [ ] **Step 6: Commit**

```bash
git add commands/run-qa.md
git commit -m "feat: add performance and mobile personas to /run-qa command"
```

---

## Task 11: Update `plugin.json` and `marketplace.json`

**Files:**
- Modify: `.claude-plugin/plugin.json`
- Modify: `.claude-plugin/marketplace.json`

- [ ] **Step 1: Read current files**

Read `.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json`.

- [ ] **Step 2: Update `plugin.json` description**

Change description to mention 5 personas:

```
"description": "QA testing pipeline with 5 personas (smoke, UX, adversarial, performance, mobile) — generate workflow docs, convert to Playwright E2E tests, run interactively or in CI. Supports quantified UX scoring with before/after binary scorecards, Next.js performance profiling, and mobile UX auditing against iOS HIG and Material Design 3 standards."
```

Add `"mobile-ux"`, `"ux-audit"`, `"scorecard"` to keywords.

- [ ] **Step 3: Update `marketplace.json` description**

Update the plugin description to match.

- [ ] **Step 4: Verify JSON syntax**

```bash
python3 -m json.tool .claude-plugin/plugin.json > /dev/null && echo "Valid"
python3 -m json.tool .claude-plugin/marketplace.json > /dev/null && echo "Valid"
```

- [ ] **Step 5: Commit**

```bash
git add .claude-plugin/plugin.json .claude-plugin/marketplace.json
git commit -m "docs: update plugin descriptions for 5 personas and UX scorecards"
```

---

## Task 12: Run validation

Verify all files pass the existing CI checks.

**Files:**
- Run: `scripts/validate-skills.sh`
- Run: `npm run validate`

- [ ] **Step 1: Run skill/agent/command validation**

```bash
./scripts/validate-skills.sh
```

Expected: all agents (6 total: smoke-tester, ux-auditor, adversarial-breaker, validation-subagent, performance-profiler, mobile-ux-auditor) and all commands pass. No errors.

- [ ] **Step 2: Run full validation suite**

```bash
npm run validate
```

Expected: markdown lint + skill validation + link check all pass.

- [ ] **Step 3: Fix any issues found**

If validation fails, fix the specific issues (likely markdown lint warnings or missing frontmatter fields) and re-run.

- [ ] **Step 4: Commit fixes if any**

```bash
git add -A
git commit -m "fix: address validation issues"
```

---

## Task Summary

| Task | Description | Dependencies |
|------|-------------|--------------|
| 1 | Delete performance-profiler skill | None |
| 2 | Create performance profiler reference | Task 1 |
| 3 | Create performance profiler agent | Task 2 |
| 4 | Create UX-auditor reference (10 categories) | None |
| 5 | Slim UX-auditor agent | Task 4 |
| 6 | Create mobile UX auditor reference | None |
| 7 | Create mobile UX auditor agent | Task 6 |
| 8 | Extract smoke-tester reference + slim agent | None |
| 9 | Extract adversarial-breaker reference + slim agent | None |
| 10 | Update `/run-qa` command | Tasks 3, 5, 7 |
| 11 | Update plugin.json + marketplace.json | None |
| 12 | Run validation | All tasks |

**Parallelizable groups:**
- Group A: Tasks 1 → 2 → 3 (performance pipeline)
- Group B: Tasks 4 → 5 (UX expansion)
- Group C: Tasks 6 → 7 (mobile pipeline)
- Group D: Tasks 8, 9, 11 (independent refactors)
- Group E: Task 10 (depends on A, B, C)
- Group F: Task 12 (depends on all)
