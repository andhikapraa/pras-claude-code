---
description: Generate E2E tests using Playwright with browser automation
model: claude-opus-4-5
---

Generate comprehensive end-to-end tests for the following feature or user flow.

## Test Target

$ARGUMENTS

---

## MCP Tools Integration

**MANDATORY:** Use these browser automation tools for E2E testing:

### Playwright MCP Tools
- `mcp__Playwright__browser_navigate` - Navigate to pages
- `mcp__Playwright__browser_snapshot` - Capture accessibility tree (preferred over screenshots)
- `mcp__Playwright__browser_click` - Click elements
- `mcp__Playwright__browser_type` - Type into inputs
- `mcp__Playwright__browser_fill_form` - Fill multiple form fields
- `mcp__Playwright__browser_evaluate` - Execute JavaScript
- `mcp__Playwright__browser_console_messages` - Capture console logs/errors
- `mcp__Playwright__browser_network_requests` - Monitor network activity
- `mcp__Playwright__browser_take_screenshot` - Visual verification

### Chrome DevTools MCP Tools (for debugging)
- `mcp__chrome-devtools__take_snapshot` - Accessibility tree snapshot
- `mcp__chrome-devtools__list_console_messages` - Console messages
- `mcp__chrome-devtools__list_network_requests` - Network inspection
- `mcp__chrome-devtools__performance_start_trace` - Performance profiling

### Context7 (for best practices)
- Fetch latest Playwright documentation and patterns

---

## E2E Testing Approach

### 1. **Test Structure**

```typescript
// tests/e2e/feature-name.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Feature Name', () => {
  test.beforeEach(async ({ page }) => {
    // Setup: navigate, authenticate if needed
  });

  test('should complete main user flow', async ({ page }) => {
    // Test implementation
  });
});
```

### 2. **Page Object Pattern**

```typescript
// tests/e2e/pages/feature.page.ts
export class FeaturePage {
  constructor(private page: Page) {}

  async navigate() {
    await this.page.goto('/feature');
  }

  async fillForm(data: FormData) {
    await this.page.fill('[data-testid="input"]', data.value);
  }

  async submit() {
    await this.page.click('[data-testid="submit"]');
  }

  async getResult() {
    return this.page.textContent('[data-testid="result"]');
  }
}
```

### 3. **Test Categories**

**Happy Path Tests**
- Complete user flows
- Expected interactions
- Success scenarios

**Error Handling Tests**
- Invalid inputs
- Network failures
- Server errors

**Edge Cases**
- Empty states
- Maximum limits
- Concurrent actions

**Accessibility Tests**
- Keyboard navigation
- Screen reader compatibility
- Focus management

### 4. **Assertions**

```typescript
// Visibility
await expect(page.locator('[data-testid="element"]')).toBeVisible();

// Text content
await expect(page.locator('h1')).toHaveText('Expected Title');

// URL
await expect(page).toHaveURL(/\/success/);

// Network
await page.waitForResponse(resp =>
  resp.url().includes('/api/endpoint') && resp.status() === 200
);

// Accessibility
const snapshot = await page.accessibility.snapshot();
expect(snapshot).toBeTruthy();
```

### 5. **Test Data Management**

```typescript
// tests/e2e/fixtures/test-data.ts
export const testUsers = {
  valid: { email: 'test@example.com', password: 'Test123!' },
  invalid: { email: 'invalid', password: '123' }
};

// Database seeding if needed
test.beforeAll(async () => {
  await seedTestData();
});

test.afterAll(async () => {
  await cleanupTestData();
});
```

---

## Output Format

### 1. **Test File Structure**

```
tests/
  e2e/
    [feature].spec.ts       # Main test file
    pages/
      [feature].page.ts     # Page object
    fixtures/
      [feature].data.ts     # Test data
```

### 2. **Playwright Config**

```typescript
// playwright.config.ts additions
export default defineConfig({
  testDir: './tests/e2e',
  use: {
    baseURL: process.env.BASE_URL || 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
  },
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
    { name: 'firefox', use: { ...devices['Desktop Firefox'] } },
    { name: 'webkit', use: { ...devices['Desktop Safari'] } },
    { name: 'mobile', use: { ...devices['iPhone 13'] } },
  ],
});
```

### 3. **npm Scripts**

```json
{
  "scripts": {
    "test:e2e": "playwright test",
    "test:e2e:ui": "playwright test --ui",
    "test:e2e:headed": "playwright test --headed",
    "test:e2e:debug": "playwright test --debug"
  }
}
```

---

## Best Practices

**DO:**
- Use `data-testid` attributes for reliable selectors
- Wait for network requests to complete
- Test actual user behavior, not implementation details
- Run tests in CI with retries
- Use trace viewer for debugging
- Test mobile viewports
- Check accessibility with every test

**DON'T:**
- Use fragile CSS selectors
- Sleep instead of waiting for conditions
- Test third-party services directly
- Skip error scenarios
- Ignore flaky tests (fix them)

---

Generate complete, production-ready E2E tests that cover the user flow thoroughly.

---
*Originally created by [Pras](https://github.com/andhikapraa/pras-claude-code)*
