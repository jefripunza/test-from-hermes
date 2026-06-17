---
name: playwright
description: "Playwright E2E testing expert: Page Object Model, visual testing, auth testing, API testing, parallel execution, CI integration."
metadata:
  hermes:
    tags: [playwright, e2e, testing, browser, automation]
---

# Playwright — E2E Testing

End-to-end testing with Playwright. Page Object Model, visual diff, auth flows.

## Expertise

- Playwright Test Runner
- Visual Testing (screenshot diff)
- Authentication Testing (login flows, session, cookies)
- API Testing (mocking, intercepting)
- Page Object Model
- Reusable fixtures
- Parallel execution
- CI/CD integration

## Project Setup

```bash
npm init playwright@latest
npx playwright install
npx playwright test
npx playwright show-report
```

## Coverage Areas

- Login / Logout
- Registration
- CRUD operations
- Permissions / Role-based access
- API integrations
- Error states
- Loading states
- Empty states
- Edge cases (network failure, timeout)

## Test Requirements

Always include:
- **Positive test cases** — happy path works
- **Negative test cases** — errors handled correctly
- **Edge cases** — empty, null, boundary values
- **Accessibility checks** — `AxeBuilder` integration

## Page Object Model

```typescript
// pages/LoginPage.ts
import { Page } from '@playwright/test'

export class LoginPage {
  constructor(private page: Page) {}

  async goto() { await this.page.goto('/login') }
  async login(email: string, password: string) {
    await this.page.fill('[data-testid="email"]', email)
    await this.page.fill('[data-testid="password"]', password)
    await this.page.click('[data-testid="submit"]')
  }
  async getErrorMessage() {
    return this.page.textContent('[data-testid="error"]')
  }
}
```

## Test Example

```typescript
import { test, expect } from '@playwright/test'
import { LoginPage } from './pages/LoginPage'

test('successful login redirects to dashboard', async ({ page }) => {
  const login = new LoginPage(page)
  await login.goto()
  await login.login('user@example.com', 'password123')
  await expect(page).toHaveURL(/\/dashboard/)
  await expect(page.locator('[data-testid="welcome"]')).toBeVisible()
})

test('invalid credentials shows error', async ({ page }) => {
  const login = new LoginPage(page)
  await login.goto()
  await login.login('wrong@email.com', 'wrongpass')
  await expect(login.getErrorMessage()).resolves.toContain('Invalid credentials')
})
```

## Visual Testing

```typescript
test('homepage matches snapshot', async ({ page }) => {
  await page.goto('/')
  await expect(page).toHaveScreenshot('homepage.png', {
    maxDiffPixels: 100,
  })
})
```

## CI Integration

```yaml
# .github/workflows/e2e.yml
- name: Run E2E tests
  run: npx playwright test --reporter=list
- name: Upload report
  if: failure()
  uses: actions/upload-artifact@v4
  with:
    name: playwright-report
    path: playwright-report/
```
