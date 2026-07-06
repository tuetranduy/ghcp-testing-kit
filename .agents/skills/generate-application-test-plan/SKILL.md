---
name: generate-application-test-plan
description: Explore a web application, generate a test plan and test scenarios. Supports 2 modes — PLAN (test plan only) and FULL (test plan + automation skeleton).
---

# Workflow: Explore Application & Generate Test Plan

> **MANDATORY SKILL:** You MUST load and carefully read the content of the **`qa-automation-engineer`** skill (at `.agents/skills/qa-automation-engineer/SKILL.md`) before starting. In addition, also refer to the **`requirements-analyzer`** and **`ui-debug-agent`** skills to support UI analysis.

This workflow helps the agent automatically explore a web application, analyze its structure, identify important modules/user flows, and generate a complete Test Plan.

## ⚠️ Execution principles

- **All output in English**
- **Do NOT guess** the app structure — you must inspect the actual DOM via MCP/browser tools
- **You must wait for user confirmation** of the scope at Step 2 before generating details
- If the user has not provided a URL → ask before starting

## The 2 Modes

| Mode | When to use | Output |
|---|---|---|
| **PLAN** (default) | The user needs to explore the app, build a test plan, identify scenarios | Modules, User Flows, Test Scenarios, Priority |
| **FULL** | The user asks for an automation skeleton too, or says "full automation suite" | Same as PLAN + Manual Test Cases + Automation Skeleton (POM + Test classes) |

> If the user says "generate full automation suite", "bootstrap automation", or asks for code → automatically switch to **FULL mode**.

## Steps

### Step 1: Receive & Explore the application (Recon)

1. Receive the application URL from the user
2. Use the **MCP browser tools** (Playwright MCP) to open the application:
   - `browser_navigate` → URL
   - `browser_resize(1920, 1080)` → desktop viewport
   - `browser_snapshot` → collect the DOM structure
3. Explore the **navigation menus**, sidebar, header to identify the main modules
4. Visit each main module in turn, using `browser_snapshot` to note:
   - Module / page name
   - The main UI components (forms, tables, buttons, modals)
   - The available actions (CRUD, search, filter, export...)
5. If the app requires login → ask the user to provide credentials or use an existing fixture

### Step 2: Analyze & Confirm the scope (Analysis — CHECKPOINT)

1. Consolidate the exploration results into a list:
   - **Discovered modules** (name, short description, number of features)
   - **Main User Flows** (Happy Path for each module)
   - **Dependencies** between modules (if any)
2. Assess a preliminary **Risk Level** for each module:
   - 🔴 **High Risk** — Core module, affects many users, complex logic
   - 🟡 **Medium Risk** — Supporting module, used frequently
   - 🟢 **Low Risk** — Rarely used module, rarely changed
3. **⏸️ STOP — Present for user review:**
   - The list of modules + risk level
   - The identified user flows
   - Ask the user: "Which modules do you want to focus on? Are there any flows to add?"
4. **Wait for the user to confirm** the scope before moving to Step 3

### Step 3: Generate Test Scenarios & Priority

1. For each module/flow confirmed by the user, generate test scenarios:
   - **Happy Path** — the main successful flow
   - **Negative Path** — wrong input, missing data, validation errors
   - **Edge Cases** — boundary values, concurrent access, empty states
2. Assign a **Priority** to each scenario based on the Risk Level:
   - **P1 (Critical)** — Core flows, regression blockers, sensitive data
   - **P2 (High)** — Main features, frequently used features
   - **P3 (Medium)** — Secondary features, UI/UX checks
   - **P4 (Low)** — Nice-to-have, cosmetic checks

### Step 4: Package the Test Plan (Output — PLAN mode)

1. Create the **artifact** `test_plan.md` with the structure:
   - **Application overview** — purpose, tech stack (if identifiable), URL
   - **List of Modules** — a table with: Module, Description, Risk Level, Number of scenarios
   - **User Flows** — describe each main flow (steps)
   - **Test Scenarios** — table: `| ID | Module | Scenario | Priority | Type (Happy/Negative/Edge) |`
   - **Automation Candidates** — mark which scenarios should be automated and why
2. If the user chooses **PLAN mode** → **STOP** here

### Step 5: Generate Manual Test Cases (FULL mode)

> Only perform this when in **FULL mode**

1. Convert the test scenarios (Step 3) into **complete manual test cases**:
   - TC ID, Module, Test Title, Pre-conditions, Test Steps, Expected Results, Test Data, Priority
2. Test Data must be **concrete** (no generic placeholders)
3. Export as a Markdown table in the artifact

### Step 6: Generate the Automation Skeleton (FULL mode)

> Only perform this when in **FULL mode**

1. Determine the automation **tech stack** (ask the user if unclear):
   - Default: Playwright + TypeScript (or Selenium + Java per preference)
2. Generate **Page Object classes** for each module:
   - Locators collected from the actual DOM (Step 1), do NOT guess
   - Clearly named interaction methods
3. Generate a **Test class skeleton** for the top-priority scenarios (P1, P2)
4. Ensure compliance with the **POM pattern** and the **locator strategy** per the rules

## Output

### PLAN mode
- Artifact `test_plan.md` including: App overview, Modules, User Flows, Test Scenarios (with Priority), Automation Candidates

### FULL mode
- All output of PLAN mode, plus:
- Manual Test Cases (Markdown table)
- Page Object classes
- Test class skeletons
- Assertions validating expected behavior