# Cross-User Sync Verification

Real-time sync assertions require checking that an action by one user is reflected in the other user's browser.

## Polling Pattern

```
1. User A performs an action (e.g., sends a message)
2. Record timestamp: sync_start = now()
3. Poll User B's browser for the expected change:
   - Check every 500ms
   - Timeout after 10 seconds (configurable per workflow)
   - Each check: use browser_snapshot or read_page to inspect User B's state
4. Record timestamp: sync_end = now()
5. Calculate: sync_time = sync_end - sync_start
6. Result:
   - If change detected: PASS (sync_time: Xms)
   - If timeout: FAIL (sync_time: >10000ms, timed out)
```

## Sync Timing Thresholds

| Category | Good | Acceptable | Slow | Failed |
|---|---|---|---|---|
| WebSocket/SSE | <500ms | 500-2000ms | 2000-5000ms | >5000ms |
| Polling-based | <2000ms | 2000-5000ms | 5000-10000ms | >10000ms |
| Database sync | <1000ms | 1000-3000ms | 3000-8000ms | >8000ms |

## When Sync Fails

If a cross-user sync assertion fails (timeout):
1. Take screenshots from BOTH browsers showing the inconsistent state
2. Check console logs in both browsers for errors
3. Check network requests for failed WebSocket/API calls
4. Create an issue task with severity "High" and sync timing data
5. Continue to next step (do not block the entire workflow)
