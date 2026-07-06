# Playwright-Specific Rules

> Apply when setting up and running automation with Playwright (TypeScript or Java).

## 1. Browser Setup (MANDATORY)

- **Debug viewport:** All UI debugging must run with a desktop viewport: **`1920x1080`**.
- **Playwright MCP — Resize required:** When using Playwright MCP to debug the UI, **ALWAYS** call `browser_resize(width=1920, height=1080)` **right after opening the browser** (after the first `browser_navigate` command). This is a mandatory step and must not be skipped.
  ```
  Mandatory order:
  1. browser_navigate(url) → open the page
  2. browser_resize(width=1920, height=1080) → set the viewport
  3. browser_snapshot() or browser_take_screenshot() → start inspecting
  ```
- **Headed mode:** Running the browser in headed mode is mandatory during test setup and debugging.
- **Headless mode:** Only allowed when:
  - The test has been debugged and passes 100% in headed mode
  - Or in the CI/CD pipeline by default

## 2. Development & Element-Finding Workflow

- Prefer using **Playwright MCP** to open the browser and interact with the target page.
- **Inspect the real DOM:** Verify and capture selectors directly from the browser DOM.
- **NEVER:**
  - Guess a locator
  - Blindly copy a locator from old code without verifying it
  - Rely on a URL / document without confirming existence on the real UI

## 3. Playwright Locator Priority Order

Playwright provides a set of user-facing semantic locators. Prefer them over CSS/XPath:

1. `getByRole()` — Best for semantic elements (button, link, heading...)
2. `getByLabel()` — Best for form fields with a label
3. `getByPlaceholder()` — Best for inputs with placeholder text
4. `getByText()` — Best for text content
5. `getByTestId()` — Best when the element has a `data-testid`
6. `locator("css")` — Fallback when no better option exists

Example:
```typescript
// Right — Semantic locator
page.getByRole('button', { name: 'Login' })
page.getByLabel('Email')
page.getByPlaceholder('Enter password')

// Wrong — Raw XPath/CSS when a semantic alternative exists
page.locator('//button[@class="btn-login"]')
page.locator('.form-input:nth-child(2)')
```

## 4. Wait Strategy

**FORBIDDEN:**
- `page.waitForTimeout()` — hard sleep
- `await new Promise(r => setTimeout(r, N))` — self-made delay
- Any way of fixing a wait time

**USE:**
- Leverage Playwright's default auto-waiting
- Web-First Assertions:
  ```typescript
  await expect(locator).toBeVisible();
  await expect(locator).toBeEnabled();
  await expect(locator).toHaveText('Success');
  await expect(page).toHaveURL(/dashboard/);
  ```
- Only use `waitForSelector()` when `expect()` cannot meet a special requirement

## 5. Test Structure

```typescript
test.describe('Module Name', () => {
  test.beforeEach(async ({ page }) => {
    // Setup: navigate, login...
  });

  test('description of the behavior to test', async ({ page }) => {
    // Arrange: initialize page objects, data
    // Act: perform the action
    // Assert: check the result
  });
});
```

- Every test block must have a **clear assertion**
- Use `test.describe` to group tests by module
- Use `beforeEach` / `afterEach` for setup / teardown
