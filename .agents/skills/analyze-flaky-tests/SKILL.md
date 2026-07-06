---
name: analyze-flaky-tests
description: Analyze unstable automation tests (flaky), identify the root cause, and fix them automatically. Supports 2 modes — ANALYZE (report only) and FIX (report + auto-fix code).
---

# Workflow: Analyze & Fix Flaky Tests

> **MANDATORY SKILL:** You MUST load and carefully read the content of the **`flaky-test-analyzer`** skill (at `.agents/skills/flaky-test-analyzer/SKILL.md`) before starting. In addition, also refer to the **`smart-locator-agent`** and **`locator-healer-agent`** skills when you need to fix or replace locators.

This workflow helps the agent automatically analyze unstable automation tests (that sometimes pass and sometimes fail), pinpoint the exact root cause, and (depending on the mode) automatically fix the code to stabilize the test.

## ⚠️ Execution principles

- **All output in English**
- **Do NOT guess** the cause — you must analyze the logs, code, and the actual DOM
- **You must wait for user confirmation** of the fix list at Step 3 before changing code (FIX mode)
- If the user has not provided a test file / error log → ask before starting
- ⚠️ After the user confirms the fix → the agent fixes and verifies on its own, and does NOT ask again during the fixing process

## The 2 Modes

| Mode | When to use | Output |
|---|---|---|
| **ANALYZE** (default) | The user wants to understand the flaky cause but does not want to change code yet | Analysis report + Proposed fixes |
| **FIX** | The user wants the agent to fix the code directly | Same as ANALYZE + Fixed code + Verification result |

> If the user says "just fix it", "fix this for me", "stabilize this test", or asks the agent to change code → automatically switch to **FIX mode**.

## Input to collect

The agent needs at least **one of the following inputs** from the user:

| Input | How to obtain it | Priority |
|---|---|---|
| **Test file path** | Provided by the user or found by the agent in the project | ⭐ Required |
| **Error log / stack trace** | Pasted by the user or collected by running the test | ⭐ Required |
| **CI/CD log** | URL or log file provided by the user | Optional |
| **Test report** (HTML/JSON) | Playwright report, Allure, TestNG report | Optional |
| **Failure history** | Number of failures / total number of runs | Optional |

## Steps

### Step 1: Gather information & Reproduce the failure (Detect & Reproduce)

1. **Read the test file**:
   - Identify the framework (Playwright / Selenium / Appium / Pytest / TestNG)
   - Note the structure: Page Objects, fixtures, helper functions
   - Flag suspicious code areas (waits, locators, assertions, setup/teardown)

2. **Run the test** to reproduce the failure (if there is no error log yet):
   - Run the test **3 times in a row** in the terminal:
     ```bash
     # Playwright
     npx playwright test <test_file> --retries=0 --reporter=list

     # Pytest
     python -m pytest <test_file> -v --count=3

     # Maven/TestNG
     mvn test -Dtest=<TestClass> -Dsurefire.rerunFailingTestsCount=0
     ```
   - Note the pattern: Which run failed? Which step failed? Is the error message the same or different?

3. **Collect evidence:**
   - Error log / stack trace from each run
   - Screenshots (if the test captures on failure)
   - Console log / Network log (if related to API calls)

### Step 2: Analyze the Root Cause (Inspect & Classify)

1. **Read the error log** and classify it against the root cause table:

| Category | Signs to recognize | Example Error |
|---|---|---|
| 🎯 **Unstable locator** | `ElementNotFound`, `No matching element`, selector contains a dynamic class/index | `Locator('.css-1a2b3c')` fails because the class changes every build |
| ⏱️ **Timing / Race condition** | `Timeout`, `Element not visible`, test passes when run slowly (debug mode) | `expect(element).toBeVisible()` times out because the animation hasn't finished |
| 📊 **Test data conflict** | `Duplicate key`, `Already exists`, fails when run in parallel | Two tests both create the user `test@email.com` |
| 🔄 **State dependency** | Passes when run independently, fails when run with the suite | Test B depends on data created by Test A |
| 🌐 **Environment / Network** | `ECONNREFUSED`, `503`, `CORS error`, fails on CI but passes locally | Slow API server, CDN timeout |
| 🖼️ **UI Animation / Transition** | `Element is not clickable`, `intercept`, fails randomly on click | Modal animation not finished, overlay covering the button |
| 📱 **Viewport / Responsive** | Element hidden, scroll doesn't reach it, fails at a different resolution | Button is outside the viewport on CI (headless 800x600) |
| 🧹 **Cleanup / Teardown** | Fails on the 2nd+ run, passes the first time | Test does not clean up data → next run has a conflict |

2. **Inspect the code** in detail, checking each anti-pattern:

   **Locator check:**
   - [ ] Does it use a dynamic class (`css-xxx`, `sc-xxx`, `MuiXxx-root`)? → ❌
   - [ ] Does it use positional xpath (`//div[3]/button[2]`)? → ❌
   - [ ] Is the locator unique? (check against the actual DOM if needed)

   **Wait strategy check:**
   - [ ] Is there a `waitForTimeout()` / `Thread.sleep()` / `time.sleep()`? → ❌
   - [ ] Is there a `waitForSelector()` when `expect()` would be enough? → ⚠️
   - [ ] Does the assertion have an appropriate timeout?

   **Test data check:**
   - [ ] Is the data hardcoded? (`test@email.com`, `user123`)
   - [ ] Is the data unique per run? (timestamp / random)
   - [ ] Is the data cleaned up after the test?

   **Test independence check:**
   - [ ] Does the test depend on the run order?
   - [ ] Does the test share state through a global variable?
   - [ ] Are setup/teardown complete?

3. **If you need to inspect the actual DOM** (locator issue):
   - Open the browser via MCP: `browser_navigate` → `browser_resize(1920, 1080)` → `browser_snapshot`
   - Compare the locator in the code against the actual DOM
   - Determine a more stable replacement locator (use the `smart-locator-agent` skill)

### Step 3: Write the report & Propose fixes (Report — CHECKPOINT)

1. **Create the artifact** `flaky_analysis.md` with the structure:

   ```markdown
   # Flaky Test Analysis Report

   ## Overview
   - **Test file:** <path>
   - **Framework:** Playwright / Selenium / ...
   - **Failure frequency:** X/Y runs
   - **Severity:** 🔴 Critical / 🟡 Medium / 🟢 Low

   ## Root Cause Analysis

   | # | Location (Line) | Category | Problem description | Severity |
   |---|---|---|---|---|
   | 1 | test.ts:45 | ⏱️ Timing | waitForTimeout(3000) instead of a smart wait | 🔴 |
   | 2 | page.ts:12 | 🎯 Locator | .css-1abc dynamic class | 🟡 |

   ## Proposed Fixes

   | # | Problem | Current code | Proposed code | Reason |
   |---|---|---|---|---|
   | 1 | Hard sleep | `waitForTimeout(3000)` | `expect(el).toBeVisible()` | Smart wait retries automatically |
   | 2 | Dynamic class | `.css-1abc` | `getByRole('button', {name: 'Submit'})` | Semantic, not dependent on CSS |
   ```

2. **⏸️ STOP — Present for user review:**
   - The list of identified root causes
   - The proposed-fix table (old code → new code)
   - Ask: "Do you agree with this analysis? Should I go ahead and fix the code, or do you just need the report?"

3. If the user chooses **ANALYZE mode** → **STOP** here
4. If the user agrees to the fix → move to Step 4

### Step 4: Automatically fix the code (Auto-Fix — FIX mode)

> Only perform this when in **FIX mode** and the user has confirmed

1. **Fix the code** in priority order (fix the most severe first):

   **Fix Locator:**
   - Use the `locator-healer-agent` skill to replace the broken locator
   - Follow the locator priority in `.agents/rules/locator_strategy.md`
   - Verify the new locator against the actual DOM before committing it into the code

   **Fix Timing:**
   ```
   ❌ page.waitForTimeout(3000)
   ✅ await expect(page.getByRole('button')).toBeVisible()

   ❌ Thread.sleep(5000)
   ✅ wait.until(ExpectedConditions.visibilityOfElementLocated(...))
   ```

   **Fix Test Data:**
   ```
   ❌ const email = "test@email.com"
   ✅ const email = `auto_${Date.now()}@test.com`
   ```

   **Fix Test Independence:**
   - Add setup/teardown to create and clean up its own data
   - Remove dependencies between test cases
   - Ensure each test creates its own preconditions

2. **Edit the files** to apply the changes
3. **Record** each change in the `flaky_analysis.md` artifact

### Step 5: Verify & Ensure stability (Verify Stability)

> Only perform this when in **FIX mode**

1. **Run the test 3 times in a row** after fixing:
   ```bash
   # Playwright
   npx playwright test <test_file> --retries=0 --repeat-each=3

   # Pytest
   python -m pytest <test_file> -v --count=3

   # Maven
   mvn test -Dtest=<TestClass> (run 3 times manually)
   ```

2. **Evaluate the result:**

   | Result | Action |
   |---|---|
   | ✅ **3/3 PASS** | Test is stable → update the artifact, report success |
   | ⚠️ **2/3 PASS** | Still flaky → return to Step 2 to analyze the failed run, keep fixing (max 3 loops) |
   | ❌ **0-1/3 PASS** | Fix in the wrong direction → roll back, re-analyze the root cause |

3. **Stability Checklist** (all must be met before finishing):
   - [ ] Locator is unique and stable across many reloads
   - [ ] No hard sleep / fixed delay
   - [ ] Test data is unique + traceable every run
   - [ ] Test is independent — not dependent on run order
   - [ ] Test passes **3+ times in a row**
   - [ ] No remaining debug log / commented code

4. **Update the artifact** `flaky_analysis.md`:
   - Add a "Results after fix" section with the run-results table
   - Mark the status: ✅ STABILIZED / ⚠️ PARTIALLY FIXED / ❌ STILL FLAKY

## Handling special situations

| Situation | How to handle it |
|---|---|
| **Test fails only on CI** | Compare CI vs local env (viewport, timezone, headless, resources). Check whether viewport `1920x1080` is set on CI |
| **Test fails when run in parallel** | Check test data isolation, shared state, database locks. Propose unique data per worker |
| **Test fails after a new deploy** | Check DOM changes, API response changes. Use `locator-healer-agent` to update locators |
| **Test fails randomly with no pattern** | Collect more data (run 10+ times), check for memory leaks, resource exhaustion |
| **Multiple tests are flaky together** | Find the common factor (shared fixture, shared page object, shared config) |
| **Test fails due to an external API** | Propose mocking/stubbing external dependencies, retry logic for API calls |

## Output

### ANALYZE mode
- Artifact `flaky_analysis.md`:
  - Test overview (file, framework, failure frequency)
  - Root Cause Analysis (classification table)
  - Detailed proposed fixes (old code → new code)
  - Stability checklist

### FIX mode
- All output of ANALYZE mode, plus:
  - The fixed code (the changed files)
  - Verification result (3 runs PASS/FAIL)
  - Final status: STABILIZED / PARTIALLY FIXED / STILL FLAKY