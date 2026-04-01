# Mobile Workflows

> Hand-authored ground truth for QA eval system.
> Last updated: 2026-04-01
> Application: TaskFlow
> Base URL: http://localhost:3000

<!-- viewport: mobile (393x852) -->

## Quick Reference

| # | Workflow | Priority | Auth | Steps |
|---|---------|----------|------|-------|
| 1 | Mobile Login and Dashboard | core | no | 6 |
| 2 | Mobile Project Creation | core | required | 8 |
| 3 | Mobile Task View | feature | required | 7 |
| 4 | Mobile Navigation Menu | edge | required | 7 |

---

## Core Workflows

## Workflow 1: Mobile Login and Dashboard

<!-- auth: no -->
<!-- priority: core -->
<!-- estimated-steps: 6 -->
<!-- viewport: mobile (393x852) -->

**Preconditions:**
- A registered user account exists with email "admin@taskflow.com" and password "password123"
- Browser viewport is set to mobile dimensions (393x852)

1. Navigate to /login — the login page in mobile viewport
2. Type "admin@taskflow.com" in the "Email" field — enter the user email
3. Type "password123" in the "Password" field — enter the user password
4. Click the "Sign In" button — submit the login form
5. Verify the dashboard heading displays "Welcome, Admin User" — confirms successful login and redirect to dashboard
6. Verify the "Menu" button text reads "Menu" — confirms the hamburger menu is rendered in mobile layout

**Postconditions:**
- User is authenticated and on the dashboard page
- The mobile hamburger menu is accessible

---

## Workflow 2: Mobile Project Creation

<!-- auth: required -->
<!-- priority: core -->
<!-- estimated-steps: 8 -->
<!-- viewport: mobile (393x852) -->

**Preconditions:**
- User is logged in as admin
- Browser viewport is set to mobile dimensions (393x852)

1. Click the "Menu" button — open the hamburger navigation menu
2. Verify the navigation menu contains "New Project" — confirms the menu is open and nav links are rendered
3. Click the "New Project" link — navigate to the create project form
4. Type "Mobile Test Project" in the "Project Name" field — fill in the project name
5. Type "Created from mobile" in the "Description" field — fill in the project description
6. Select "Public" from the "Visibility" dropdown — set the project visibility
7. Click the "Create" button — submit the new project form
8. Verify the heading displays "Mobile Test Project" — confirms the project was created with the correct name

**Postconditions:**
- A new project named "Mobile Test Project" exists
- The project detail page is displayed

---

## Feature Workflows

## Workflow 3: Mobile Task View

<!-- auth: required -->
<!-- priority: feature -->
<!-- estimated-steps: 7 -->
<!-- viewport: mobile (393x852) -->

**Preconditions:**
- User is logged in as admin
- A project "Alpha Project" with tasks exists (including "Set up authentication" with done status)
- Browser viewport is set to mobile dimensions (393x852)

1. Navigate to /dashboard — the main dashboard page
2. Click the "Alpha Project" link — navigate to the project detail page
3. Verify the task list contains "Set up authentication" — confirms the task list renders correctly in mobile viewport
4. Verify the status badge contains "done" — confirms status badges display in the mobile layout
5. Click the "Set up authentication" link — tap the task to navigate to detail page
6. Verify the heading displays "Set up authentication" — confirms navigation to the correct task detail page
7. Verify the comments section contains "Looks good, merging now" — confirms task comments are displayed in mobile layout

**Postconditions:**
- User is on the task detail page for "Set up authentication"
- Task details and comments are visible in mobile viewport

---

## Edge Case Workflows

## Workflow 4: Mobile Navigation Menu

<!-- auth: required -->
<!-- priority: edge -->
<!-- estimated-steps: 7 -->
<!-- viewport: mobile (393x852) -->

**Preconditions:**
- User is logged in as admin
- Browser viewport is set to mobile dimensions (393x852)

1. Navigate to /dashboard — the main dashboard page
2. Click the "Menu" button — open the hamburger navigation menu
3. Verify the navigation menu contains "Dashboard" — confirms the dashboard link is accessible in the menu
4. Verify the navigation menu contains "New Project" — confirms the new project link is accessible in the menu
5. Click the "Menu" button — close the hamburger navigation menu
6. Verify the "Dashboard" link is no longer visible — confirms the menu closed successfully
7. Verify the dashboard heading displays "Welcome, Admin User" — confirms the page content is still accessible after menu close

**Postconditions:**
- The hamburger menu opens and closes correctly
- All navigation links are accessible through the mobile menu
