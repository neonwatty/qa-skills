# Example Multi-User Workflow

## Workflow 1: Create Room and Invite Another User

### Personas
- User A: Room host (authenticated, admin role)
- User B: Invited member (authenticated, member role)

### Prerequisites
- Both users have existing accounts
- App running at localhost:3000
- Both users logged in with separate browser sessions

### Steps
1. [User A] Navigate to /rooms
2. [User A] Click "Create Room" button
3. [User A] Type "Test Room" in room name field
4. [User A] Click "Create" -> room is created, redirected to /rooms/[id]
5. [User A] Verify: room page loads with "Test Room" title and empty member list
6. [User A] Click "Invite" button in the toolbar
7. [User A] Type User B's email in the invite field
8. [User A] Click "Send Invite" -> invitation sent confirmation appears
9. [User B] Verify: notification badge appears in the header (cross-user sync)
10. [User B] Click the notification bell icon
11. [User B] Verify: sees "You've been invited to Test Room" notification
12. [User B] Click "Accept" on the invitation
13. [User B] Verify: redirected to /rooms/[id], room content visible
14. [User A] Verify: User B's name appears in the member list (cross-user sync)
15. [User A] Verify: member count updates from 1 to 2
