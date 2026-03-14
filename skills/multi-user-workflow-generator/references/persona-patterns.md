# Persona Patterns & Workflow Categories

## Workflow Categories

When exploring the codebase, look for these common multi-user workflow types:

| Category | What to Look For | Example Workflow |
|----------|-----------------|------------------|
| Authentication & session isolation | Login, signup, sessions | Two users logged in simultaneously see their own data |
| Room/space creation and joining | Room create, join, invite | User A creates room, User B joins via link |
| Real-time content sync | Insert, update, delete + subscription | User A adds item, User B sees it appear in real time |
| Real-time state sync | Queue advance, toggles, counters | User A advances queue, User B sees position update |
| Social features | Friend requests, blocking, invitations | User A sends friend request, User B accepts |
| Notification delivery | In-app, push, email notifications | User B receives notification when User A mentions them |
| Permission boundaries | Role checks, RLS, visibility | User A (admin) sees controls User B (member) does not |
| Conflict resolution | Simultaneous edits, last-write-wins | Both users edit same item, system resolves conflict |
| Leave/rejoin flows | Disconnect, leave room, rejoin | User A leaves room, User B sees them leave, User A rejoins |
| Invite-to-signup | Invite link, new user onboarding | User A invites email, recipient signs up and joins |

## Multi-User Journey Complexity Levels

### Two-User Flows (User A + User B)
- Authentication isolation (two accounts, separate sessions)
- Invitation and acceptance
- Content sharing from one user to another
- Real-time sync between two users
- Permission boundaries (owner vs viewer)

### Multi-User Flows (3+ concurrent users)
- Room/space with multiple participants
- Broadcast updates seen by all members
- Role-based visibility within groups
- Moderation and admin actions

### Edge Case Flows (unusual but critical)
- Simultaneous edits / conflict resolution
- Offline user reconnects and sees missed updates
- User leaves room/space while others remain
- Blocked user attempts interaction
- Invite-to-signup (new user onboarding via invitation link)

## Persona Prefix Convention

- Always prefix every step with `[User A]`, `[User B]`, etc.
- Define each persona in the Personas section at the top of each workflow
- Include role (admin, member, guest, anonymous) and authentication state
- Use consistent persona names throughout a workflow
