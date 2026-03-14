# HTML Audit Report Template

The audit report MUST be HTML and MUST embed screenshots from the execution phase.

## Required HTML Structure

```html
<!-- Required sections: -->
<h1>Multi-User Workflow Audit Report</h1>
<p>Date: [timestamp] | Environment: [URL]</p>
<p>User A: [identity/role] | User B: [identity/role]</p>

<!-- Summary table -->
<table>
  <tr><th>#</th><th>Workflow</th><th>Status</th><th>Steps</th><th>Sync Time</th><th>Notes</th></tr>
  <!-- One row per workflow with PASS/FAIL/SKIP badge -->
</table>

<!-- Sync timing summary -->
<h2>Real-Time Sync Performance</h2>
<table>
  <tr><th>Assertion</th><th>Sync Time (ms)</th><th>Status</th></tr>
  <!-- One row per cross-user assertion -->
</table>

<!-- Per-workflow detail sections -->
<h2>Workflow N: [Name]</h2>
<p>Status: PASS/FAIL/SKIP</p>
<h3>Steps</h3>
<ol>
  <li>[User A] Step description - PASS/FAIL
    <br><img src="screenshots/multi-user-audit/wfNN-stepNN-userA.png" style="max-width:800px; border:1px solid #ddd; border-radius:8px; margin:8px 0;">
  </li>
  <li>[User B] Step description - PASS/FAIL
    <br><img src="screenshots/multi-user-audit/wfNN-stepNN-userB.png" style="max-width:800px; border:1px solid #ddd; border-radius:8px; margin:8px 0;">
  </li>
</ol>
```

## Requirements

- Every workflow section MUST include `<img>` tags referencing screenshots saved during execution
- Use relative paths: `screenshots/multi-user-audit/wfNN-stepNN-userX.png`
- Include side-by-side screenshot comparisons for cross-user assertions (User A view next to User B view)
- Style with clean design, professional appearance, app accent color
- Update the HTML file incrementally after EACH workflow so partial results are always viewable

## Findings Log Format

Append to `.claude/plans/multi-user-workflow-findings.md` after each workflow:

```markdown
---
### Workflow [N]: [Name]
**Timestamp:** [ISO datetime]
**Status:** Passed/Failed/Partial
**Personas:** User A ([role]), User B ([role])

**Steps Summary:**
- Step 1 [User A]: [Pass/Fail] - [brief note]
- Step 2 [User B]: [Pass/Fail] - [brief note]
...

**Cross-User Sync Results:**
- [Assertion description]: [Pass/Fail] - [sync time in ms]
- Average sync time: [ms]
- Max sync time: [ms]
- Sync failures: [count]

**Issues Found:**
- [Issue description] (Severity: High/Med/Low) (User: A/B/Cross-user)

**UX/Design Notes:**
- [Observation]

**Technical Problems:**
- [Problem] (include console errors if any)

**Feature Ideas:**
- [Idea]

**Screenshots:** [list of screenshot paths captured from both browsers]
```
