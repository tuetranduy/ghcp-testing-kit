# General Rules for QA Automation

> Apply to every automation testing task, regardless of framework (Playwright, Selenium, Appium).

## 1. Architecture & Framework

- The **Page Object Model (POM)** is mandatory.
- Separate clearly:
  - **Page classes:** Declare locators + UI interaction methods
  - **Test classes:** Contain the test logic + assertions
  - **Test data:** Kept separate from functional code (JSON, DataProvider, Utils)
- Assertions belong only in Test classes, NOT in Page classes.

## 2. Test Data Generation

- All fields requiring uniqueness (Email, Username, Customer Code...) **must be generated dynamically**, not hardcoded.
- Use UUIDs, timestamps, or a Faker library.
- Data must be **traceable** — looking at the DB should immediately reveal which test created it:
  ```
  Format: [prefix]_[testName]_[timestamp]_[random]
  Example: auto_createCustomer_20260402_A3F2@test.com
  ```
- Support parallel execution: each test method has its own distinct data, no conflicts.

## 3. Code Quality

- No duplicated logic — create helper methods for repeated actions.
- Code must be simple, readable, and maintainable.
- Before delivering code:
  - Remove all `console.log`, `System.out.println`, and `print()` added during debugging
  - Remove commented-out code (`//`, `/* */`)
  - Remove unused locators / variables (unused code)

## 4. File & Directory Management

- Do NOT automatically delete source files without confirming with the user.
- Check the existing directory structure before creating new files — avoid duplicates.
- Place files in the correct directory per the project architecture (see `plan/automation/0_project_architecture`).

## 5. Naming Conventions

### Java

| Component        | Convention                                  | Example                                 |
| ---------------- | ------------------------------------------- | --------------------------------------- |
| Page class       | PascalCase + `Page` suffix                  | `LoginPage.java`, `CartPage.java`       |
| Test class       | PascalCase + `Test` suffix                  | `LoginTest.java`, `CartTest.java`       |
| Test method      | Start with `test` + behavior description    | `testLoginWithValidCredentials()`       |
| Locator variable | lowerCamelCase + element-descriptive suffix | `loginButton`, `usernameInput`          |
| Utils class      | PascalCase + function description           | `DataGenerator.java`, `WaitHelper.java` |

### TypeScript / Playwright

| Component        | Convention                     | Example                                 |
| ---------------- | ------------------------------ | --------------------------------------- |
| Page class       | PascalCase + `Page` suffix     | `LoginPage.ts`, `CartPage.ts`           |
| Test file        | kebab-case + `.spec.ts`        | `login.spec.ts`, `cart.spec.ts`         |
| Test block       | `test('behavior description')` | `test('login successfully')`            |
| Locator variable | lowerCamelCase or readonly     | `readonly loginButton`                  |
| Utils            | PascalCase or kebab-case       | `DataGenerator.ts`, `data-generator.ts` |

## 6. Assertions

- Every test case **MUST** have at least 1 assertion at the end.
- It is good practice to have interspersed assertions at important steps.
- Assertions must clearly describe the expected behavior:
  ```java
  // Java/TestNG
  Assert.assertTrue(dashboardPage.isDisplayed(), "Dashboard must be displayed after logging in");
  ```
  ```typescript
  // Playwright
  await expect(page.getByText('Login successful')).toBeVisible();
  ```

## 7. Test Independence

- Each test case must be **independent** — it must not depend on the result of another test.
- Clear setup/teardown (`@BeforeMethod/@AfterMethod` or `beforeEach/afterEach`).
- Do not share state between test methods.