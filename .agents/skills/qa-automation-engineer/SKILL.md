---
name: qa-automation-engineer
description: Skill that helps the agent perform QA automation testing tasks including generating test cases, automation scripts, API tests, and locators, analyzing flaky tests, and creating test data.
---

# QA Automation Engineer

## Description

This skill enables the agent to assist with software testing and automation tasks.

The agent can:

- Generate manual test cases from requirements
- Generate test automation scripts from test cases or UI flows
- Generate API tests from Swagger/OpenAPI specifications
- Explore applications and discover test scenarios
- Generate automation frameworks
- Generate test data
- Analyze flaky tests
- Generate stable locators
- Generate requirements from website analysis

This skill is designed for modern QA workflows and automation development.

---

# When to Use

Use this skill when the user asks about:

- Test automation
- Manual testing
- Automation frameworks
- API testing
- UI testing
- Test data generation
- Flaky test debugging
- Locator generation
- Requirements analysis from website
- Jira integration (fetch requirements, push test results)
- Xray test management

Typical prompts include:

- Generate test cases from requirement
- Generate Selenium automation from test case
- Generate automation from UI steps
- Generate API tests from Swagger
- Generate regression suite → _(redirect to `generate-application-test-plan` or `generate-manual-testcases-rbt`)_
- Generate test data
- Analyze flaky test
- Generate locator for element
- Generate requirements from website

---

# Workflow Routing

When the user request matches a specific task, select the appropriate workflow skill from `.agents/skills/`.

### Generate test cases from requirements

> **Delegate:** This task belongs to the **`rbt-manual-testing`** skill — not `qa-automation-engineer`.

Use workflow: `generate-testcases-from-requirements` (QUICK mode) or `generate-manual-testcases-rbt` (FULL RBT mode).

Triggers when user asks:

- generate test cases → **delegate to `rbt-manual-testing` (QUICK mode)**
- write manual test cases → **delegate to `rbt-manual-testing` (QUICK mode)**
- test scenarios from requirement → **delegate to `rbt-manual-testing` (QUICK mode)**
- generate full test cases / 6-step process → **delegate to `rbt-manual-testing` (FULL RBT mode)**

---

### Generate automation from manual test case

Use workflow: `generate-automation-from-testcases`

Triggers when user asks:

- convert test case to automation
- generate Selenium automation
- generate Playwright automation from test case

---

### Generate automation from UI steps

Use workflow: `generate-automation-from-ui-flow`

Triggers when user asks:

- automate this UI flow
- generate automation from steps
- run UI steps and generate Selenium script

---

### Generate API tests

Use workflow: `generate-api-tests-from-swagger`

Triggers when user provides:

- Swagger URL
- OpenAPI specification

---

### Generate test data

Use workflow: `generate-test-data`

Triggers when user asks:

- generate test data
- generate boundary test data

---

### Analyze cross-module feature & generate combinatorial matrix

Use workflow: `generate-cross-module-test-plan`

> Workflow for **complex features that pass through multiple sequential modules**. Generates a Data Flow Map + a multi-dimensional combinatorial matrix (Pairwise / Business-critical / Full Cartesian).

Triggers when user asks:

- analyze a cross-module feature
- test multiple linked modules
- generate a combinatorial matrix
- test a feature with many combined conditions
- analyze multi-module feature
- pairwise testing
- multi-dimensional decision table

---

### Generate combinatorial test data (multi-module pipeline)

Use workflow: `generate-combinatorial-test-data`

> Generates test data for a combinatorial matrix. Supports 2 modes: **GENERATE** (offline generation) and **PIPELINE** (running for real in the browser across N modules).

Triggers when user asks:

- generate data for a combinatorial matrix
- create test data for a combinatorial matrix
- run a pipeline to create data across multiple modules
- generate combinatorial test data
- set up data for a cross-module test

---

### Generate regression suite

> **No dedicated workflow.** Use `generate-application-test-plan` (PLAN mode) or `generate-manual-testcases-rbt` (FULL RBT) depending on the input.

Triggers when user asks:

- create regression test suite
- generate regression scenarios

---

### Generate automation framework

> **Delegate:** This task uses the **`framework-architect`** skill to design the framework.

Use workflow: `generate-automation-framework`

Triggers when user asks:

- create automation framework
- design Selenium framework
- design Playwright framework
- design Appium framework
- scaffold automation project
- design a new framework

---

### Explore application and generate test plan

Use workflow: `generate-application-test-plan`

> This workflow has **2 modes**: PLAN (default — test plan only) and FULL (test plan + automation skeleton).
> When the user asks for a "full automation suite" or "bootstrap automation" → automatically select FULL mode.

Triggers when user asks:

- explore application
- discover test scenarios
- generate test plan
- generate full automation suite
- bootstrap automation for project

---

### Analyze flaky tests

Use workflow: `analyze-flaky-tests`

Triggers when user asks:

- why is this test flaky
- analyze unstable automation

---

### Generate stable locators

Use workflow: `generate-locator`

Triggers when user asks:

- generate locator for this element
- find stable selector
- create automation locator

---

### Generate requirements from website

Use workflow: `generate-requirements-from-website`

Triggers when user asks:

- generate requirements from website
- analyze website module and create requirements
- extract user stories from web page

---

### Analyze requirement document

> **Delegate:** This task uses the **`requirements-analyzer`** skill to analyze requirement documents.

Use workflow: `analyze-requirement-document`

> The workflow only **analyzes** the requirement — it does NOT generate test cases. The output is a detailed analysis document that includes: AC breakdown, dependencies, ambiguities, risks.

Triggers when user asks:

- analyze a requirement document
- review requirements / analyze this ticket
- analyze a Jira ticket / requirement
- find ambiguities in a requirement
- analyze requirement / review requirement document

---

### Fetch requirements from Jira

Use workflow: `fetch-jira-requirements`

Triggers when user asks:

- fetch jira requirements
- get requirement from jira
- get jira ticket
- import user stories from jira

---

### Import test results to Xray

Use workflow: `import-test-results-xray`

Triggers when user asks:

- push test results to xray
- push test results to xray
- import test execution to jira
- upload playwright results to xray

---

# Automation Framework

Default automation stack:

- **Language:** Java
- **UI automation:** Selenium WebDriver or Playwright
- **Test framework:** TestNG
- **API automation:** REST Assured
- **Mobile automation:** Appium
- **Design pattern:** Page Object Model (POM)

---

# Locator Strategy

## Selenium Locator Priority

1. `id`
2. `data-testid`
3. `name`
4. `css selector`
5. `xpath` (last resort)

Avoid fragile locators such as auto-generated class names or positional xpaths.

## Playwright Locator Priority

1. `getByRole()`
2. `getByLabel()`
3. `getByPlaceholder()`
4. `getByText()`
5. `getByTestId()`
6. `css selector`
7. `xpath` (last resort)

Avoid fragile selectors such as dynamic class names.

> **Note:** For detailed locator rules, refer to `.agents/rules/locator_strategy.md`.

---

# Rules References

The agent MUST also follow the detailed rules defined in `.agents/rules/`:

- [automation_rules.md](../../rules/automation_rules.md) — General automation best practices
- [locator_strategy.md](../../rules/locator_strategy.md) — Detailed locator selection rules
- [playwright_rules.md](../../rules/playwright_rules.md) — Playwright-specific rules
- [selenium_rules.md](../../rules/selenium_rules.md) — Selenium-specific rules
- [appium_rules.md](../../rules/appium_rules.md) — Appium mobile automation rules

---

# References

The agent may consult additional documentation in the `references/` folder:

- `PROJECT_CONTEXT.md` — Project domain, tech stack, key modules
- `TEST_STRATEGY.md` — Testing objectives, scope, execution plan
- `PROMPT_TEMPLATES.md` — Reusable prompt templates for common QA tasks

External references:

- `plans/automation/project_architecture/README.md` — Repository structure & project architecture
- `.agents/rules/delivery_checklist.md` — Quality checklist / Definition of Done

---

# Output

Depending on the request, the agent may return:

- Manual test cases (structured format)
- Automation scripts (Java/TypeScript)
- API tests (REST Assured)
- Locator recommendations
- Test data (structured, randomized, traceable)
- Automation framework design
- Requirements documents

Automation outputs should include:

- Page Object classes
- Test classes
- Assertions validating expected behavior
- Clean, readable, maintainable code (no debug logs, no commented code)
