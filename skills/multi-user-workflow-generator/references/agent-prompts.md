# Exploration Agent Prompts

These are the full prompts for the three parallel Explore agents spawned in Phase 2 of the multi-user workflow generator.

## Agent 1: Auth & User Roles

```
Task tool parameters:
- subagent_type: "Explore"
- model: "sonnet"
- prompt: |
    You are exploring a web application to find all authentication, session management, and user role patterns that affect multi-user interactions.

    ## What to Find

    1. **Authentication Flows**
       - Search for auth middleware (e.g., `middleware.ts`, `auth.ts`, route guards)
       - Find login, signup, logout, and password-reset flows
       - Identify token handling (JWT, session cookies, refresh tokens)
       - Note OAuth/SSO providers if present

    2. **User Role Definitions**
       - Grep for role enums or constants: `role`, `admin`, `host`, `guest`, `owner`, `member`, `anonymous`
       - Find RLS (Row Level Security) policies in SQL or ORM config
       - Identify permission checks: `canEdit`, `isOwner`, `hasPermission`, `authorize`
       - Note any role hierarchy (e.g., admin > moderator > member > guest)

    3. **Session Management**
       - Find session creation and validation logic
       - Identify multi-session handling (same user, multiple devices)
       - Note session expiry, invalidation, and refresh patterns
       - Check for impersonation or "act as" features

    4. **User Identity & Profiles**
       - Find user model/schema with relevant fields
       - Identify display name, avatar, and presence indicators
       - Note user-to-user relationship models (friends, contacts, teams)

    ## Search Patterns
    - Files: `**/auth*`, `**/middleware*`, `**/session*`, `**/user*`, `**/role*`, `**/permission*`
    - Grep: `createUser`, `signIn`, `signUp`, `signOut`, `getSession`, `currentUser`, `requireAuth`
    - Grep: `RLS`, `policy`, `row_level`, `check_access`, `permission`
    - Grep: `role`, `admin`, `host`, `guest`, `anonymous`, `owner`, `member`

    ## Return Format

    ```
    ## Auth & User Roles Report

    ### Authentication Flows
    | Flow | Entry Point | Method | Notes |
    |------|-------------|--------|-------|

    ### User Roles
    | Role | Permissions | Defined In |
    |------|------------|------------|

    ### Session Management
    - Session type: [cookie/JWT/etc.]
    - Multi-device support: [yes/no]
    - Key files: [list]

    ### Permission Boundaries
    | Resource | Owner Can | Member Can | Guest Can | Anonymous Can |
    |----------|-----------|------------|-----------|---------------|
    ```
```

## Agent 2: Real-Time & Shared State

```
Task tool parameters:
- subagent_type: "Explore"
- model: "sonnet"
- prompt: |
    You are exploring a web application to find all real-time features, shared state, and synchronization patterns between multiple users.

    ## What to Find

    1. **Real-Time Subscriptions**
       - Search for Supabase Realtime: `supabase.channel`, `.on('postgres_changes'`, `subscribe()`
       - Find WebSocket connections: `new WebSocket`, `socket.io`, `ws://`, `wss://`
       - Identify Server-Sent Events: `EventSource`, `text/event-stream`
       - Note polling patterns: `setInterval` + fetch, SWR/React Query refetch intervals

    2. **Shared Database Tables**
       - Find tables/collections that multiple users read and write
       - Identify which entities are scoped per-user vs shared
       - Note any shared rooms, spaces, channels, or workspaces
       - Check for collaborative documents or shared lists

    3. **Optimistic Updates & Conflict Resolution**
       - Find optimistic UI patterns: update UI before server confirms
       - Identify conflict resolution: last-write-wins, merge, CRDT
       - Note any retry or rollback logic on failed mutations
       - Check for version numbers or timestamps on records

    4. **Presence & Live Indicators**
       - Find presence systems: online/offline status, typing indicators
       - Identify live cursors, live avatars, or "who's viewing" features
       - Note any activity feeds or real-time notifications

    ## Search Patterns
    - Files: `**/realtime*`, `**/socket*`, `**/channel*`, `**/subscribe*`, `**/sync*`, `**/presence*`
    - Grep: `supabase`, `realtime`, `subscribe`, `channel`, `broadcast`, `presence`
    - Grep: `WebSocket`, `socket.io`, `EventSource`, `onmessage`
    - Grep: `optimistic`, `conflict`, `merge`, `CRDT`, `version`
    - Grep: `setInterval`, `refetch`, `polling`, `stale`

    ## Return Format

    ```
    ## Real-Time & Shared State Report

    ### Real-Time Channels
    | Channel/Subscription | Table/Event | Purpose | File |
    |---------------------|-------------|---------|------|

    ### Shared State
    | Entity | Scope | Read By | Written By | Sync Method |
    |--------|-------|---------|------------|-------------|

    ### Optimistic Updates
    | Action | Optimistic Behavior | Rollback On Failure | File |
    |--------|--------------------|--------------------|------|

    ### Presence Features
    - Online indicators: [yes/no, details]
    - Typing indicators: [yes/no, details]
    - Live cursors: [yes/no, details]
    - Activity feed: [yes/no, details]
    ```
```

## Agent 3: Cross-User Interactions

```
Task tool parameters:
- subagent_type: "Explore"
- model: "sonnet"
- prompt: |
    You are exploring a web application to find all cross-user interaction features -- places where one user's actions affect another user's experience.

    ## What to Find

    1. **Invitation & Onboarding Flows**
       - Find invite systems: email invites, invite links, invite codes
       - Identify join flows: accept/decline invitation, request to join
       - Note invite-to-signup: invited user creates an account
       - Check for referral or sharing mechanisms

    2. **Social Features**
       - Find friend/follow systems: `follow`, `friend`, `connect`, `request`
       - Identify blocking/privacy: `block`, `mute`, `restrict`, `privacy`
       - Note user search or discovery features
       - Check for user profiles visible to other users

    3. **Notification Systems**
       - Find in-app notifications: toast, notification center, badges
       - Identify push notifications: FCM, APNs, web push
       - Note email notifications: transactional emails, digests
       - Check notification preferences and opt-out

    4. **Content Sharing & Collaboration**
       - Find content sharing: share buttons, public links, embed codes
       - Identify collaborative editing: multiple cursors, comments, suggestions
       - Note chat or messaging features: DMs, group chat, threads
       - Check for reactions, likes, votes, or ratings across users

    5. **Room/Space Management**
       - Find room creation: `createRoom`, `createSpace`, `createChannel`
       - Identify join/leave flows: `joinRoom`, `leaveRoom`, `addMember`
       - Note room settings: visibility, capacity, moderation
       - Check for room roles: host, moderator, participant, viewer

    ## Search Patterns
    - Files: `**/invite*`, `**/notification*`, `**/share*`, `**/chat*`, `**/message*`, `**/room*`, `**/social*`
    - Grep: `invite`, `invitation`, `joinLink`, `inviteCode`, `referral`
    - Grep: `follow`, `friend`, `block`, `mute`, `privacy`
    - Grep: `notification`, `notify`, `toast`, `badge`, `push`, `fcm`
    - Grep: `share`, `collaborate`, `comment`, `react`, `like`, `vote`
    - Grep: `createRoom`, `joinRoom`, `leaveRoom`, `addMember`, `removeMember`
    - Grep: `chat`, `message`, `thread`, `DM`, `direct_message`

    ## Return Format

    ```
    ## Cross-User Interactions Report

    ### Invitation Flows
    | Flow | Mechanism | New User Signup? | File |
    |------|-----------|-----------------|------|

    ### Social Features
    | Feature | Actions | Privacy Controls | File |
    |---------|---------|-----------------|------|

    ### Notification Systems
    | Trigger | In-App | Push | Email | File |
    |---------|--------|------|-------|------|

    ### Content Sharing
    | Content Type | Share Methods | Permissions | File |
    |-------------|---------------|-------------|------|

    ### Room/Space Management
    | Entity | Create | Join | Leave | Roles | File |
    |--------|--------|------|-------|-------|------|
    ```
```

## Post-Agent Task Updates

After each agent returns, update its task:
```
TaskUpdate:
- taskId: [explore task ID]
- status: "completed"
- metadata: {
    "authFlowsFound": [count],           # For auth agent
    "rolesFound": [count],               # For auth agent
    "realtimeChannelsFound": [count],    # For real-time agent
    "sharedEntitiesFound": [count],      # For real-time agent
    "crossUserFeaturesFound": [count],   # For cross-user agent
    "notificationTypesFound": [count],   # For cross-user agent
    "summary": "[brief summary of findings]"
  }
```
