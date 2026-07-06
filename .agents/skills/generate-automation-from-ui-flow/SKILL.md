---
name: generate-automation-from-ui-flow
description: Execute a UI flow directly in the browser, collect locators from the real DOM, and generate automation scripts. Supports Playwright, Selenium, Appium.
---

# Workflow: Generate Automation from a UI Flow

> **MANDATORY SKILL:** You MUST load and carefully read the skill **`ui-debug-agent`** (at `.agents/skills/ui-debug-agent/SKILL.md`) before starting. Also refer to the skill **`smart-locator-agent`** for generating stable locators and **`qa-automation-engineer`** for the shared automation rules.

This workflow helps the agent **execute directly** a sequence of UI actions in a real browser, collect locators from the real DOM, and generate complete automation scripts — all in a single automated flow, without an existing manual test case.

## ⚠️ Execution principles

- **All output in English**
- **NEVER guess a locator** — it must be taken from the real DOM using the MCP browser tools
- **You must run each UI step in a real browser** before generating code
- **Desktop viewport 1920×1080** for all UI debugging
- ⚠️ **Rule E3:** When a test FAILs → read the log → analyze → fix → rerun. Do NOT ask the user

## How is this different from `generate-automation-from-testcases`?

| | `from_testcases` | `from_ui_flow` (this workflow) |
|---|---|---|
| **Input** | An existing manual test case file | UI steps described in words or a URL + actions |
| **Approach** | Read TCs → inspect UI → generate code | **Run live in the browser** → collect locators → generate code |
| **When to use** | A test case document already exists | No TCs yet, you only know "go to this page, click that thing" |

## Inputs to collect

The agent needs at least **one of the following inputs** from the user:

| Input | Example | Priority |
|---|---|---|
| **URL + described UI steps** | "Go to https://example.com, log in, create a new user" | ⭐ Most common |
| **URL + recording/video** | User provides a video of the actions | Optional |
| **URL + screenshots** | User provides screenshots of each step | Optional |
| **URL only** | "Automate this page's login flow" | The agent discovers it |

If the user has not provided enough → ask:
- Application URL?
- Description of the flow to automate (step by step)?
- Credentials if login is needed?
- Desired framework? (default: Playwright + TypeScript)

## Steps

### Step 1: Intake & Preparation (Setup)

1. **Parse the UI steps** from the user's input:
   - Convert the spoken description into a structured list of steps:
     ```
     Step 1: Navigate to https://example.com/login
     Step 2: Enter username "admin@test.com"
     Step 3: Enter password "***"
     Step 4: Click Login button
     Step 5: Verify dashboard is displayed
     ```

2. **Confirm the tech stack** with the user (if not yet clear):

   | Framework | Language | When to use |
   |---|---|---|
   | **Playwright** | TypeScript | Default for web automation |
   | **Playwright** | Python | When the user uses a Python stack |
   | **Selenium** | Java | When the user requests Java/Selenium |
   | **Appium** | Java | Mobile app automation |

3. **Create the `task.md` artifact** to track progress:
   ```markdown
   # UI Flow Automation Progress
   - [ ] Step 1: Preparation — parse UI steps
   - [ ] Step 2: Run the UI flow in the browser — collect locators
   - [ ] Step 3: Generate Page Objects + Test scripts
   - [ ] Step 4: Run tests + Auto-heal
   ```

### Step 2: Run the UI Flow in the Browser & Collect Locators (Live Recon)

> ⚡ This is the **most important** step — it distinguishes this workflow from the others.

1. **Open the browser with MCP** and navigate to the URL:
   ```
   browser_navigate → URL
   browser_resize → 1920 × 1080
   browser_wait_for → page load complete
   browser_snapshot → collect the initial DOM
   ```

2. **Execute each step** in the list; for each step:

   ```
   a. browser_snapshot → read the DOM, identify the element to interact with
   b. Determine the best locator (per locator priority)
   c. Execute the action (click / type / select / hover)
   d. browser_snapshot → confirm the action result
   e. Record it in the locator collection table
   ```

3. **Locator Collection table** (record after each step):

   | Step | Action | Element | Primary Locator | Fallback Locator | Verified |
   |---|---|---|---|---|---|
   | 1 | Navigate | — | — | — | ✅ |
   | 2 | Type | Username input | `getByLabel('Email')` | `#email` | ✅ |
   | 3 | Type | Password input | `getByLabel('Password')` | `#password` | ✅ |
   | 4 | Click | Login button | `getByRole('button', {name: 'Login'})` | `button[type=submit]` | ✅ |
   | 5 | Assert | Dashboard title | `getByRole('heading', {name: 'Dashboard'})` | `.dashboard-title` | ✅ |

4. **Locator Priority** (follow `.agents/rules/locator_strategy.md`):

   **Playwright:**
   `getByRole()` → `getByLabel()` → `getByPlaceholder()` → `getByText()` → `getByTestId()` → CSS → XPath

   **Selenium:**
   `id` → `data-testid` → `name` → CSS selector → XPath

   **Appium:**
   `accessibility-id` → `id` → `name` → `xpath` (relative)

5. **Handle situations while running the UI:**

   | Situation | How to handle |
   |---|---|
   | Element not found | `browser_snapshot` again → check the DOM → try another locator |
   | Page not fully loaded | `browser_wait_for` text/element → retry |
   | Modal/popup appears | Handle the popup first → continue the flow |
   | Redirect/navigation | `browser_snapshot` again on the new page |
   | Scrolling needed | `browser_evaluate` → scrollIntoView |
   | Login required | Ask the user for credentials or use an existing fixture |
   | CAPTCHA / 2FA | Notify the user — cannot be automated |

6. **Screenshot evidence** — capture at key milestones:
   - After a successful login
   - After completing the main flow
   - When encountering an error/unexpected state

### Step 3: Generate Automation Scripts (Code Generation)

1. **Generate Page Object classes** from the locator collection:

   **Playwright TypeScript:**
   ```typescript
   // src/pages/login.page.ts
   import { Page, Locator } from '@playwright/test';

   export class LoginPage {
     readonly page: Page;
     readonly emailInput: Locator;
     readonly passwordInput: Locator;
     readonly loginButton: Locator;

     constructor(page: Page) {
       this.page = page;
       this.emailInput = page.getByLabel('Email');
       this.passwordInput = page.getByLabel('Password');
       this.loginButton = page.getByRole('button', { name: 'Login' });
     }

     async login(email: string, password: string) {
       await this.emailInput.fill(email);
       await this.passwordInput.fill(password);
       await this.loginButton.click();
     }
   }
   ```

   **Selenium Java:**
   ```java
   // src/main/java/.../pages/LoginPage.java
   public class LoginPage extends BasePage {
     @FindBy(id = "email")
     private WebElement emailInput;

     @FindBy(id = "password")
     private WebElement passwordInput;

     @FindBy(css = "button[type='submit']")
     private WebElement loginButton;

     public void login(String email, String password) {
       waitAndType(emailInput, email);
       waitAndType(passwordInput, password);
       waitAndClick(loginButton);
     }
   }
   ```

2. **Generate the Test class:**
   - Import the Page Objects
   - Structure: **Arrange → Act → Assert**
   - Clear assertions with descriptive messages
   - Unique + traceable test data (use timestamp/random)

3. **Code generation principles:**
   - Locators MUST come from Step 2 (verified against the DOM) — DO NOT guess
   - Do not hardcode test data (read credentials from env/config)
   - Do not use `waitForTimeout()` / `Thread.sleep()` — only smart waits
   - Method names describe user behavior, not the DOM actions
   - Each page → 1 file, each test → 1 file

### Step 4: Run Tests & Self-Heal (Execution & Auto-Heal)

1. **Run the tests** in the terminal:
   ```bash
   # Playwright TS
   npx playwright test <test_file> --headed

   # Playwright Python
   python -m pytest <test_file> --headed

   # Selenium Java
   mvn test -Dtest=<TestClass>
   ```

2. **Track the results** by checking the terminal output:
   - If **PASS** → move on to verify stability
   - If **FAIL** → enter the Auto-Heal loop:

   ```
   WHILE the test FAILs (up to 5 rounds):
     1. Read the error log → identify the failing step
     2. Classify the error:
        - Wrong locator → open the browser MCP, re-inspect the DOM, replace the locator
        - Timing issue → add a smart wait or adjust the assertion timeout
        - Wrong page state → check the flow, add a wait for navigation
        - Test data conflict → generate new (unique) data
     3. Fix the code by editing the file
     4. Rerun the test
   ```

3. **Verify stability** — run the test **2 consecutive times** PASSing:
   ```bash
   # Playwright
   npx playwright test <test_file> --repeat-each=2

   # Pytest
   python -m pytest <test_file> --count=2
   ```

4. **⚠️ Rule E3:** Do NOT ask the user while fixing errors. Only ask when:
   - The URL is blocked / requires a captcha
   - Business logic conflicts (the expected behavior is unclear)
   - The 5 auto-heal rounds are exhausted and it still fails

### Step 5: Cleanup & Delivery

1. **Code cleanup** (mandatory before delivery):
   - [ ] Remove `console.log()` / `print()` / debug logs
   - [ ] Remove unused locators
   - [ ] Remove commented-out code
   - [ ] No more `waitForTimeout()` / `Thread.sleep()`
   - [ ] No redundant imports

2. **Update the `task.md` artifact** with the results:
   ```markdown
   ## Results
   - ✅ Pages created: LoginPage, DashboardPage
   - ✅ Tests created: login.spec.ts
   - ✅ Test status: 2/2 PASS (stable)
   - 📊 Locators collected: 8 elements, all verified
   ```

3. **Report** to the user:
   - List of files created
   - Number of PASS/FAIL tests
   - The locator collection table (for the user's reference)
   - Known limitations (if any)

## Output

- **Artifact `task.md`** — progress checklist + results
- **Page Object classes** — 1 file per page/screen, locators verified from the DOM
- **Test classes** — complete automation scripts, tested and PASSing
- **Locator Collection table** — all collected elements + primary/fallback locators
- **Evidence screenshots** — captured at key milestones