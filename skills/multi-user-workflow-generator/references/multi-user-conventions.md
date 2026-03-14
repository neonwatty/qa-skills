# Multi-User Anti-Patterns & Conventions

When generating workflows, watch for these common multi-user testing anti-patterns and ensure your workflows avoid them.

## Synchronization Anti-Patterns

| Anti-Pattern | Issue | Better Alternative |
|---|---|---|
| No real-time sync verification | Workflows don't verify other users see updates | Add cross-user Verify steps after every mutation |
| Single-user-only testing | Workflows only test one user at a time | Always test with 2+ personas simultaneously |
| Assuming instant sync | Steps assume updates appear immediately | Include Wait steps and verify sync timing |
| No sync failure testing | Missing tests for what happens when sync fails | Add scenarios for network interruption during sync |

## Permission Anti-Patterns

| Anti-Pattern | Issue | Better Alternative |
|---|---|---|
| Missing permission boundaries | No tests for what User B should NOT see | Add negative assertions for unauthorized access |
| Only testing happy-path roles | Only admin role tested | Test every role: admin, member, guest, anonymous |
| No role escalation tests | Missing tests for privilege changes | Test what happens when roles change mid-session |

## Session Anti-Patterns

| Anti-Pattern | Issue | Better Alternative |
|---|---|---|
| Hardcoded user IDs | Tests break when data changes | Use dynamic values from API responses |
| Shared session state | Tests assume clean state | Include setup/teardown for each persona |
| No offline/reconnect testing | Missing network interruption scenarios | Include disconnect/reconnect verification steps |
| No concurrent mutation tests | Only sequential interactions tested | Test simultaneous actions by multiple users |

## Notification Anti-Patterns

| Anti-Pattern | Issue | Better Alternative |
|---|---|---|
| No notification verification | Workflows skip checking if notifications arrive | Verify notification delivery for every cross-user action |
| Only in-app notifications | Push and email notifications untested | Test all configured notification channels |
| No notification preference tests | Missing tests for opt-out behavior | Verify users can control their notification preferences |
