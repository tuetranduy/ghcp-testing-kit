# AI-DRIVEN AUTOMATION TESTING FRAMEWORK

**Goal:**
Use AI to automate the process of building an Automation Framework, designing the POM, and generating high-quality code — maintainable and CI/CD friendly.

## 📌 Core Principles

1. **AI DOM Recon First:** The AI must use the browser MCP (Playwright/Selenium) to open the browser itself and inspect the real DOM. NEVER guess locators.
2. **POM Architecture:** Every script follows the Page Object Model — clearly separating Pages and Tests.
3. **Smart Waits & Stability:** No hard sleeps (`Thread.sleep`, `waitForTimeout`). Use auto-waiting only.
4. **Deterministic Data:** Test data must be unique and traceable, never hardcoded.
5. **Self-fix Loop:** The AI runs the tests itself → if they fail → it reads the logs, fixes the code, and re-runs → until they PASS.

---

## 🚀 6-Step Workflow + Step 0 (Setup)

| Step  | Name                 | Purpose                                               | Skill                                            |
| ----- | -------------------- | ----------------------------------------------------- | ------------------------------------------------ |
| **0** | Project Architecture | Set up the standard directory architecture            | `qa-automation-engineer`                         |
| **1** | Context & Role-play  | Establish the role + tech stack                       | `qa-automation-engineer`                         |
| **2** | Analysis & UI Recon  | The AI opens the browser itself and collects locators | `qa-automation-engineer` + `ui-debug-agent`      |
| **3** | POM Design           | Design the Page Object classes                        | `qa-automation-engineer`                         |
| **4** | Test Data Strategy   | Generate the Data Generator class                     | `qa-automation-engineer` + `test-data-generator` |
| **5** | Script Generation    | Generate the test script + auto-run + auto-fix        | `qa-automation-engineer`                         |
| **6** | Review & Refactoring | Clean code + CI/CD readiness                          | `qa-automation-engineer`                         |

*(Each step corresponds to one subfolder containing a `README.md` + `prompt.txt`.)*

---

## 🎯 Execution Strategy

### Approach 1: Sequential (Manual Control)

Use each `prompt.txt` file manually (01 → 06):
- **Suitable when:** You only need the AI to do one specific step (e.g., "Only generate the POM, no test script needed").
- **Or when:** The project is large and you want fine-grained control over each module.

### Approach 2: One-Click Auto Workflow (Recommended)

Use the generate-automation-from-testcases workflow and attach your Test Cases:
- The agent handles it all: Read TCs → Open Browser → Collect Locators → Generate POM → Generate Script → Run Tests → Fix Bugs → until they PASS.
- **Advantage:** Fully automated after a single command.

---

## 📋 Reference Rules

- `.agents/rules/automation_rules.md` — POM, Data, Naming conventions
- `.agents/rules/locator_strategy.md` — Locator priority order
- `.agents/rules/playwright_rules.md` — Playwright-specific rules
- `.agents/rules/selenium_rules.md` — Selenium-specific rules
- `.agents/rules/appium_rules.md` — Appium-specific rules

## 📁 Quick Start

See the `QUICK_START.md` file in this directory to get started quickly.
