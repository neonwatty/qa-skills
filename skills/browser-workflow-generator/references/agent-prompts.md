# Agent Prompts Reference

Detailed prompts for agents spawned during workflow generation.

## Phase 2: Exploration Agents

### Agent 1 - Routes & Navigation

```
Task tool parameters:
- subagent_type: "Explore"
- model: "sonnet"
- prompt: |
    You are exploring a web application to find all routes and navigation patterns.

    ## What to Find

    1. **All Routes/Pages**
       - Search for router configuration (React Router, Next.js pages, Vue Router, etc.)
       - Find all page/view components
       - Identify URL patterns and parameters

    2. **Navigation Patterns**
       - Find navigation menus, sidebars, headers
       - Identify breadcrumbs, tabs, or other nav UI
       - Map how users move between pages

    3. **Entry Points**
       - Find the main entry URL
       - Identify deep links or bookmarkable URLs
       - Note any authentication-gated routes

    ## Return Format

    ```
    ## Routes & Navigation Report

    ### All Routes
    | Route | Component | Purpose | Auth Required |
    |-------|-----------|---------|---------------|

    ### Navigation Structure
    - Primary nav: [description]
    - Secondary nav: [description]

    ### User Flow Map
    - Home -> [possible destinations]
    - [Page] -> [possible destinations]
    ```
```

### Agent 2 - Components & Features

```
Task tool parameters:
- subagent_type: "Explore"
- model: "sonnet"
- prompt: |
    You are exploring a web application to find all interactive UI components.

    ## What to Find

    1. **Interactive Components**
       - Buttons, links, clickable elements
       - Form inputs (text, select, checkbox, etc.)
       - Modals, dialogs, drawers
       - Drag-drop areas, toolbars, menus

    2. **Major Features**
       - Identify the app's core features
       - Find feature entry points in the UI
       - Note feature-specific components

    3. **Component Patterns**
       - Identify reusable component patterns
       - Note any component libraries used (MUI, Radix, etc.)
       - Find data-testid or accessibility attributes

    ## Return Format

    ```
    ## Components & Features Report

    ### Major Features
    | Feature | Entry Point | Key Components |
    |---------|-------------|----------------|

    ### Interactive Components by Page
    #### [Page Name]
    - Buttons: [list]
    - Forms: [list]
    - Other: [list]

    ### Component Patterns
    - UI library: [if any]
    - Common patterns: [list]
    ```
```

### Agent 3 - State & Data

```
Task tool parameters:
- subagent_type: "Explore"
- model: "sonnet"
- prompt: |
    You are exploring a web application to understand its data model and user actions.

    ## What to Find

    1. **Data Model**
       - Find state management (Redux, Zustand, Context, etc.)
       - Identify main data entities/types
       - Note data relationships

    2. **User Actions (CRUD)**
       - What can users create?
       - What can users read/view?
       - What can users update/edit?
       - What can users delete?

    3. **API & Persistence**
       - Find API call patterns
       - Identify endpoints used
       - Note localStorage/sessionStorage usage

    ## Return Format

    ```
    ## State & Data Report

    ### Data Entities
    | Entity | Properties | CRUD Operations |
    |--------|------------|-----------------|

    ### User Actions
    - Create: [list of things users can create]
    - Read: [list of things users can view]
    - Update: [list of things users can modify]
    - Delete: [list of things users can remove]

    ### API Patterns
    - Base URL: [if found]
    - Key endpoints: [list]
    ```
```

## Phase 4: UX Research Agent

```
Task tool parameters:
- subagent_type: "general-purpose"
- model: "sonnet"
- prompt: |
    You are researching web UX conventions for a workflow generator.

    ## Page Types to Research
    [Include list of page types identified from Phase 2/3, e.g.:]
    - Login page
    - Dashboard
    - Settings page
    - List/table view
    - Detail page
    - Onboarding flow

    ## Your Task

    For each page type:

    1. **Search for reference examples** using WebSearch:
       - "web app [page type] design Dribbble"
       - "best SaaS [page type] UI examples"
       - "[well-known web app] [page type] screenshot"

    2. **Visit 2-3 reference examples** to understand conventions

    3. **Document UX conventions** for each page type

    ## Return Format

    For each page type, return:
    ```
    ### Page: [Page Type]
    **Reference Examples:** [Apps/sites compared]
    **Expected Web Conventions:**
    - [Convention 1]
    - [Convention 2]
    - [Convention 3]
    **Anti-patterns to flag:**
    - [Anti-pattern 1 - why it's wrong for web]
    - [Anti-pattern 2]
    ```

    ## Example Output

    ### Page: Dashboard
    **Reference Examples:** Linear, Notion, Figma
    **Expected Web Conventions:**
    - Top navigation bar with logo, search, and user menu
    - Sidebar for section navigation (collapsible on smaller screens)
    - Cards or widgets for key metrics
    - Hover states on all interactive elements
    - Keyboard shortcuts for power users (with visible hints)
    - Responsive layout that works from 320px to 2560px+
    **Anti-patterns to flag:**
    - Bottom tab bar (mobile pattern)
    - Pull-to-refresh gesture requirement
    - No hover states on buttons/links
    - URLs that don't reflect current view state
```
