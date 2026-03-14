# Multi-Context Playwright Patterns

Multi-user tests differ from single-user tests in several critical ways.

## Persona-to-Context Mapping

Each persona in the workflow becomes a `browser.newContext()` with its own `context.newPage()`. This gives each persona independent cookies, localStorage, and authentication state.

```typescript
// Each persona gets their own isolated browser context
contextA = await browser.newContext()  // Host / Admin / Sender
contextB = await browser.newContext()  // Guest / Member / Receiver
pageA = await contextA.newPage()
pageB = await contextB.newPage()
```

## Cross-Context Assertions

After one user acts, assert the result from the other user's page. This validates real-time sync, notifications, and shared state updates.

```typescript
// User A creates something
await pageA.locator('[data-testid="create-btn"]').click()

// User B should see it appear (with timeout for propagation)
await expect(pageB.getByText('New item')).toBeVisible({ timeout: 10000 })
```

## Real-Time Sync Timeouts

WebSocket, SSE, or polling-based real-time features need extended timeouts. Use `{ timeout: 10000 }` (10 seconds) for assertions that depend on cross-user data propagation.

## Skipping Unreachable Flows

Flows requiring real external services (email delivery, push notifications, SMS) should use `test.skip` with a clear explanation.

## Anti-Patterns to Avoid

### 1. Sharing Contexts Between Personas
```typescript
// BAD: Both users share the same context (shared cookies/auth)
const page1 = await context.newPage()
const page2 = await context.newPage()

// GOOD: Each user gets their own context
contextA = await browser.newContext()
contextB = await browser.newContext()
pageA = await contextA.newPage()
pageB = await contextB.newPage()
```

### 2. Missing Sync Timeouts on Cross-User Assertions
```typescript
// BAD: Default timeout too short for real-time propagation
await expect(pageA.getByText('2 watching')).toBeVisible()

// GOOD: Extended timeout for WebSocket/SSE/polling propagation
await expect(pageA.getByText('2 watching')).toBeVisible({ timeout: 10000 })
```

### 3. Forgetting to Close Contexts in afterEach
```typescript
// BAD: Context leak causes flaky tests and resource exhaustion
test.afterEach(async () => {
  // nothing here
})

// GOOD: Always close all contexts
test.afterEach(async () => {
  await contextA.close()
  await contextB.close()
})
```

### 4. UI-Driven Precondition Setup for Every Test
```typescript
// BAD: Slow, fragile, repeats UI flows for setup
test('guest joins', async () => {
  // 20 lines of UI steps to create user, login, create party...
  await pageA.goto('/register')
  await pageA.fill('[name="email"]', 'host@example.com')
  // ... many more UI steps ...
})

// GOOD: Use API helpers for fast precondition setup
test('guest joins', async () => {
  const party = await createPartyViaAPI('host-token', 'Test Party')
  await pageB.goto(`/join/${party.code}`)
  // Test only the guest join flow
})
```

### 5. Testing External Service Delivery
```typescript
// BAD: Trying to assert on email/push/SMS delivery
await expect(pageB.getByText('Email received')).toBeVisible()

// GOOD: Skip flows requiring real external services
test.skip('guest receives email notification', async () => {
  // SKIP: Requires real email delivery service
  // Test email content via unit tests instead
})
```

### 6. Hardcoded User Credentials in Test Body
```typescript
// BAD: Credentials scattered across tests
test('host creates', async () => {
  await pageA.fill('[name="email"]', 'host@example.com')
  await pageA.fill('[name="password"]', 'secret123')
})

// GOOD: Centralized auth via API helpers or test fixtures
test.beforeEach(async ({ browser }) => {
  await loginViaAPI(pageA, TEST_USERS.host.email, TEST_USERS.host.password)
  await loginViaAPI(pageB, TEST_USERS.guest.email, TEST_USERS.guest.password)
})
```

### 7. Sequential Page Actions Without Waiting
```typescript
// BAD: Race condition between users
await pageA.locator('[data-testid="send-btn"]').click()
await pageB.locator('[data-testid="message"]').click()  // May not exist yet

// GOOD: Wait for cross-user propagation before acting
await pageA.locator('[data-testid="send-btn"]').click()
await expect(pageB.locator('[data-testid="message"]')).toBeVisible({ timeout: 10000 })
await pageB.locator('[data-testid="message"]').click()
```
