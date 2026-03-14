# Agent Prompts

Full prompts for agents spawned during the iOS workflow generation process.

## Phase 2: Exploration Agents

### Agent 1 - Pages & Navigation

```
Task tool parameters:
- subagent_type: "Explore"
- model: "sonnet"
- prompt: |
    You are exploring a web application to find all pages and navigation patterns.
    This app will be tested in Safari on iOS Simulator.

    ## What to Find

    1. **All Pages/Routes**
       - Search for router configuration (React Router, Next.js pages, Vue Router, etc.)
       - Find all page/view components
       - Identify URL patterns and parameters

    2. **Navigation Patterns**
       - Find navigation components (tabs, sidebars, menus)
       - Identify modal/sheet presentations
       - Map how users move between pages
       - Note any gesture-based navigation

    3. **Entry Points**
       - Find the main entry URL (likely localhost:5173 or similar)
       - Identify deep links or bookmarkable URLs
       - Note any authentication-gated routes

    ## Return Format

    ```
    ## Pages & Navigation Report

    ### All Pages
    | Route | Component | Purpose | Auth Required |
    |-------|-----------|---------|---------------|

    ### Navigation Structure
    - Primary nav: [tab bar / sidebar / etc.]
    - Secondary nav: [description]
    - Modal presentations: [list]

    ### Base URL
    - Development: [URL]
    - Production: [URL if found]
    ```
```

### Agent 2 - UI Components & Interactions

```
Task tool parameters:
- subagent_type: "Explore"
- model: "sonnet"
- prompt: |
    You are exploring a web application to find all interactive UI components.
    This app will be tested on iOS Safari, so note touch-friendliness.

    ## What to Find

    1. **Interactive Components**
       - Buttons, links, tappable elements
       - Form inputs (text, select, toggle, date picker)
       - Modals, sheets, drawers
       - Drag-drop areas, lists

    2. **Touch Interactions**
       - Find gesture handlers (swipe, pinch, long press)
       - Note touch event listeners
       - Identify touch target sizes (should be 44pt+)

    3. **Component Patterns**
       - Identify component libraries used
       - Note iOS-style vs web-style components
       - Find accessibility labels/attributes

    ## Return Format

    ```
    ## UI Components Report

    ### Interactive Components by Page
    #### [Page Name]
    - Buttons: [list with approximate sizes]
    - Forms: [inputs and their types]
    - Gestures: [swipe/pinch handlers if any]

    ### Touch Considerations
    - Components with small touch targets: [list]
    - Gesture-dependent interactions: [list]

    ### Component Library
    - Using: [library name or "custom"]
    - iOS-native feel: [yes/no/partial]
    ```
```

### Agent 3 - Data & State

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
       - Note localStorage/sessionStorage/cookies usage
       - Find offline/caching strategies

    ## Return Format

    ```
    ## Data & State Report

    ### Data Entities
    | Entity | Properties | CRUD Operations |
    |--------|------------|-----------------|

    ### User Actions
    - Create: [list]
    - Read: [list]
    - Update: [list]
    - Delete: [list]

    ### Persistence
    - API base: [URL if found]
    - Local storage keys: [list]
    - Offline support: [yes/no]
    ```
```

## Phase 4: HIG Research Agent

```
Task tool parameters:
- subagent_type: "general-purpose"
- model: "sonnet"
- prompt: |
    You are researching iOS UX conventions for a workflow generator.
    The web app being tested should feel like a native iOS app.

    ## Screen Types to Research
    [Include list of screen types identified from Phase 2/3, e.g.:]
    - Login screen
    - Settings page
    - List view
    - Detail view
    - Onboarding flow
    - Search interface

    ## Your Task

    For each screen type:

    1. **Search for reference examples** using WebSearch:
       - "iOS [screen type] design Dribbble"
       - "best iOS [screen type] UI examples"
       - "[well-known iOS app like Airbnb/Spotify] [screen type] screenshot"
       - "iOS Human Interface Guidelines [component]"

    2. **Visit 2-3 reference examples** to understand iOS conventions

    3. **Document iOS UX conventions** for each screen type

    ## Return Format

    For each screen type, return:
    ```
    ### Screen: [Screen Type]
    **Reference Examples:** [iOS apps compared]
    **Expected iOS Conventions:**
    - [Convention 1 - specific to iOS]
    - [Convention 2]
    - [Convention 3]
    **Anti-patterns to flag (things that feel "webby"):**
    - [Anti-pattern 1 - why it breaks iOS feel]
    - [Anti-pattern 2]
    ```

    ## Example Output

    ### Screen: Login Screen
    **Reference Examples:** Airbnb, Spotify, Instagram
    **Expected iOS Conventions:**
    - Large, centered logo or app name
    - Email/password fields using native iOS text field styling
    - Social login buttons with standard iOS button height (50pt)
    - "Forgot Password" as text link, not button
    - Sign up CTA clearly visible but secondary to login
    - Keyboard avoidance - form should scroll when keyboard appears
    **Anti-patterns to flag:**
    - Web-style dropdown for country code (should use iOS picker)
    - Tiny touch targets on social buttons (<44pt)
    - Hamburger menu visible on login screen
    - Material Design styled inputs
```
