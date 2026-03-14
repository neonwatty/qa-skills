# API Helper Functions

Multi-user tests often require complex preconditions (user accounts, shared resources). Use API helpers to set up state quickly instead of driving the UI for every prerequisite.

## User Registration

```typescript
async function createUserViaAPI(email: string, password: string) {
  const response = await fetch('/api/auth/register', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ email, password }),
  })
  return response.json()
}
```

## User Login (Token-Based)

```typescript
async function loginViaAPI(page: Page, email: string, password: string) {
  // TODO: Replace with your app's auth mechanism
  const response = await fetch('/api/auth/login', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ email, password }),
  })
  const { token } = await response.json()
  await page.evaluate((t) => localStorage.setItem('token', t), token)
}
```

## Resource Creation

```typescript
async function createPartyViaAPI(hostToken: string, partyName: string) {
  const response = await fetch('/api/parties', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${hostToken}`,
    },
    body: JSON.stringify({ name: partyName }),
  })
  return response.json() // { id, code, name }
}
```

## Test User Constants

```typescript
const TEST_USERS = {
  host: { email: 'host@example.com', password: 'testpass123' },
  guest: { email: 'guest@example.com', password: 'testpass123' },
  admin: { email: 'admin@example.com', password: 'testpass123' },
  member: { email: 'member@example.com', password: 'testpass123' },
}
```

## Usage in beforeEach

```typescript
test.beforeEach(async ({ browser }) => {
  contextA = await browser.newContext()
  contextB = await browser.newContext()
  pageA = await contextA.newPage()
  pageB = await contextB.newPage()

  // Centralized auth via API helpers
  await loginViaAPI(pageA, TEST_USERS.host.email, TEST_USERS.host.password)
  await loginViaAPI(pageB, TEST_USERS.guest.email, TEST_USERS.guest.password)
})
```
