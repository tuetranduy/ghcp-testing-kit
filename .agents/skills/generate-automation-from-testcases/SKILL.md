---
name: generate-automation-from-testcases
description: Convert manual test cases into automation scripts autonomously using the 6-step AI-RBT Framework and Copilot tools.
---

# Workflow: Generate Automation Scripts from Manual Test Cases

> **MANDATORY SKILLS:** You MUST load and carefully read the following skills before starting:
> - **`qa-automation-engineer`** (`.agents/skills/qa-automation-engineer/SKILL.md`) — Shared automation rules + workflow routing
> - **`ui-debug-agent`** (`.agents/skills/ui-debug-agent/SKILL.md`) — Inspect the DOM, collect locators
> - **`smart-locator-agent`** (`.agents/skills/smart-locator-agent/SKILL.md`) — Generate stable locators
> - **`test-data-generator`** (`.agents/skills/test-data-generator/SKILL.md`) — Generate unique, traceable test data

This workflow helps the agent read a manual test case file provided by the user, open a browser to inspect the UI, collect real locators, generate complete automation scripts (POM + Test), run the tests, and self-heal errors until they PASS.

## ⚠️ Execution principles

- **Role:** The agent acts as a Senior Automation Engineer — following Clean Code + POM
- **All output in English**
- **NEVER guess a locator** — you must inspect the real DOM using the MCP browser tools
- **Desktop viewport 1920×1080** for all UI debugging
- ⚠️ **Rule E3 (CRITICAL):** When a test FAILs → read the log → analyze → fix the code → rerun. **Do NOT ask the user during error fixing.** Only ask when there is a conflicting business rule or after exhausting 5 auto-heal rounds
- **Artifact `task.md`** — you MUST create it to track progress across the 6 steps

## How is this different from `generate-automation-from-ui-flow`?

| | `from_testcases` (this workflow) | `from_ui_flow` |
|---|---|---|
| **Input** | Structured manual test case file (MD/Excel/JSON) | UI steps described in words or just a URL |
| **Existing TCs** | ✅ Already available — read and convert | ❌ Not available — the agent discovers them |
| **Approach** | Read TCs → inspect UI to verify → generate code | Run live in the browser → collect → generate code |

## Inputs to collect

| Input | How to obtain | Priority |
|---|---|---|
| **Test case file** (MD/Excel/JSON/URL) | User provides a path or URL | ⭐ Required |
| **Application URL** | User provides it or it is in the TCs | ⭐ Required |
| **Credentials** (if login is needed) | User provides them or use an existing fixture | Optional |
| **Tech stack** | User specifies it or detect from the project | Optional |

If the user has not provided enough → ask before starting.

## Steps

### Step 1: Initialize, Analyze & Plan (Context & Analysis)

1. **Read the test case file** provided by the user:
   - Local file → read the file
   - URL (Google Sheets, Confluence, etc.) → fetch the webpage
   - Identify the format: Markdown table, Excel, JSON, or free-form text

2. **Parse the test cases** and extract:
   - The list of TCs (ID, Title, Steps, Expected Results, Test Data, Priority)
   - The pages/screens the TCs go through
   - Pre-conditions (login, setup data, navigate...)
   - Dependencies between TCs (if any)

3. **Determine the tech stack** (if not yet clear):

   | Framework | Language | Runner | When to choose |
   |---|---|---|---|
   | Playwright | TypeScript | Playwright Test | Default for web |
   | Playwright | Python | Pytest | When the project uses Python |
   | Selenium | Java | TestNG | When the user requests Java |
   | Appium | Java | TestNG | Mobile app |

4. **Create the `task.md` artifact** to track progress:
   ```markdown
   # Automation Generation Progress
   - [x] Step 1: Analyze test cases
   - [ ] Step 2: UI recon (MCP Recon)
   - [ ] Step 3: Design the POM
   - [ ] Step 4: Prepare test data
   - [ ] Step 5: Generate automation scripts
   - [ ] Step 6: Run tests + Auto-heal

   ## Test Cases to Automate
   | TC ID | Title | Pages | Priority | Status |
   |---|---|---|---|---|
   | TC01 | Successful login | LoginPage, DashboardPage | P1 | ⏳ |
   | TC02 | Login with wrong password | LoginPage | P1 | ⏳ |
   ```

### Step 2: Autonomous UI Recon with MCP

1. **Open the browser** with MCP and navigate through the test case steps:
   ```
   browser_navigate → application URL
   browser_resize → 1920 × 1080
   browser_wait_for → page load complete
   browser_snapshot → collect the DOM
   ```

2. **For each page in the test cases**, do:
   - `browser_snapshot` → read the accessibility tree
   - Identify all elements to interact with (inputs, buttons, links, dropdowns...)
   - Collect the best locator for each element (per the priority in the `smart-locator-agent` skill)
   - Verify the locator by attempting interaction (`browser_click`, `browser_type`)

3. **Record into the Locator Collection table:**

   | Page | Element | Action | Primary Locator | Fallback Locator | Verified |
   |---|---|---|---|---|---|
   | LoginPage | Email input | Type | `getByLabel('Email')` | `#email` | ✅ |
   | LoginPage | Password input | Type | `getByLabel('Password')` | `#password` | ✅ |
   | LoginPage | Login button | Click | `getByRole('button', {name: 'Login'})` | `button[type=submit]` | ✅ |
   | DashboardPage | Welcome text | Assert | `getByRole('heading', {name: /Welcome/})` | `.welcome-header` | ✅ |

4. **Handle situations:**

   | Situation | How to handle |
   |---|---|
   | URL blocked / requires VPN | Notify the user |
   | Login required | Use an existing fixture or ask the user for credentials |
   | Element not found | Snapshot again → try another locator → notify the user if the DOM changed |
   | CAPTCHA / 2FA | Notify the user — cannot be automated |
   | Dynamic content / SPA | `browser_wait_for` a specific text before snapshotting |

5. **NEVER guess a selector** — every locator must be verified against the real DOM.

### Step 3: Design the POM (Page Object Model Architecture)

1. **Determine the list of Page classes** to create:
   - Each page/screen in the test flow → 1 Page class
   - Consider creating a `BasePage` if the project does not have one

2. **Generate the Page Object classes** by creating the files:

   **Structure of each Page class:**
   ```
   - Locators (declared at the top of the class — from Step 2)
   - Constructor (receives the page/driver instance)
   - Action methods (describe user behavior, not the DOM)
   - Verification methods (check state/text after an action)
   ```

   **Principles:**
   - Method names describe behavior: `login()`, `fillRegistrationForm()`, not `clickButton()`
   - No hardcoded waits — only smart waits
   - Locators come from Step 2 (already verified) — DO NOT guess
   - Return `this` or the next page object for method chaining (if appropriate)

3. **Check the current project structure:**
   - If the project already has pages/ → generate the file into the correct folder
   - If it is a new project → create the structure per the `framework-architect` skill
   - Do not create duplicates — check whether the page already exists before creating a new one

### Step 4: Prepare Data (Test Data Strategy)

1. **Analyze the test data** from the test cases:
   - Which data must be **unique per run** (email, username, ID) → generate random + traceable
   - Which data is **fixed** (URL, config values) → read from env/config
   - Which data needs **multiple sets** (data-driven) → create an external file (JSON/YAML)

2. **Generate test data utilities** (using the `test-data-generator` skill):
   ```
   Format: <prefix>_<testName>_<timestamp>
   Example:
   - Email:    auto_login_1712049200@test.com
   - Username: auto_user_1712049200
   - Code:     TC_REG_1712049200
   ```

3. **Sensitive data** (credentials):
   - Read from env variables or a config file
   - **Do NOT hardcode** in the test code
   - **Do NOT read .env directly** (security rule)

### Step 5: Generate Automation Scripts (Test Classes)

1. **Create test classes** — each test case or group of related TCs → 1 test file:

   **Structure of each test:**
   ```
   Setup (Arrange):
   - Initialize page objects
   - Prepare test data
   - Navigate to the page under test
   - Login (if needed, via a fixture)

   Execution (Act):
   - Perform the steps from the test case
   - Call methods from the Page Objects

   Verification (Assert):
   - Assert the result against the expected results from the TC
   - Clear assertion message, easy to debug on failure
   ```

2. **Mandatory assertions:**
   - Each TC MUST have at least 1 assertion
   - The assert message describes clearly: `"Expected dashboard to show after login"`
   - Use soft assertions when checking multiple points
   - Suitable timeout (do not leave the default too short)

3. **Code principles:**
   - No `waitForTimeout()` / `Thread.sleep()` — only smart waits
   - No inline locators in tests — locators live in the Page class
   - Clean imports — no unused imports
   - Independent tests — no dependence on run order
   - Cleanup/teardown if the test creates data

### Step 6: Test Execution & Auto-Heal (Execution & Auto-Heal — RULE E3)

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

   **If PASS:**
   - Run again **one more time** to confirm stability
   - Update `task.md`: TC status → ✅ PASS
   - Clean up debug logs, commented code

   **If FAIL → Enter the Auto-Heal loop:**

   ```
   WHILE the test FAILs (up to 5 rounds):
     1. Read the error log / stack trace → identify the failing step
     2. Classify the error:

        | Error | Action |
        |---|---|
        | Element not found | Open MCP → snapshot → verify/replace the locator |
        | Click intercepted | Wait for the overlay to disappear → retry the click |
        | Timeout | Increase the timeout or add a wait condition |
        | Assertion fail | Check expected value vs actual → update the assertion |
        | Navigation error | Check the URL, redirect, page load |
        | Test data conflict | Generate new unique data |
        | Import/compile error | Fix imports, check the class name |

     3. Fix the code by editing the file
     4. Rerun the test
     5. Log to task.md: "Round 2: Fixed locator XYZ → PASS"
   ```

3. **⚠️ Rule E3 — DO NOT ASK THE USER while fixing errors.** You may only ask when:
   - Business logic conflicts (the TC says A but the app shows B)
   - The server/app is not accessible
   - The 5 auto-heal rounds are exhausted and it still fails

4. **Verify stability** — the test must PASS **2 consecutive times**:
   ```bash
   # Playwright
   npx playwright test <test_file> --repeat-each=2 --retries=0
   ```

### Step 7: Cleanup & Delivery

1. **Code cleanup** (mandatory):
   - [ ] Remove `console.log()` / `print()` / temporary debug logs
   - [ ] Remove unused locators
   - [ ] Remove commented-out code
   - [ ] No more `waitForTimeout()` / `Thread.sleep()`
   - [ ] No more hardcoded test data (email, password)
   - [ ] Clean imports — no unused imports

2. **Update the `task.md` artifact** with the final results:
   ```markdown
   ## Results
   | TC ID | Title | Status | Stability | Notes |
   |---|---|---|---|---|
   | TC01 | Successful login | ✅ PASS | 2/2 stable | — |
   | TC02 | Login with wrong password | ✅ PASS | 2/2 stable | — |
   | TC03 | Register an account | ⚠️ SKIP | — | Requires CAPTCHA |

   ## Files Created
   - src/pages/login.page.ts
   - src/pages/dashboard.page.ts
   - src/tests/login.spec.ts
   ```

3. **Report** to the user:
   - Total: X TC PASS / Y TC FAIL / Z TC SKIP
   - List of files created/edited
   - Known issues / limitations
   - The Locator Collection table (reference)

## Output

- **Artifact `task.md`** — progress checklist + test run results
- **Page Object classes** — 1 file per page, locators verified from the DOM
- **Test classes** — complete automation scripts, PASSing stably
- **Test data utilities** — generators for unique + traceable data
- **Locator Collection table** — all elements + primary/fallback locators
- **Results report** — PASS/FAIL/SKIP summary
