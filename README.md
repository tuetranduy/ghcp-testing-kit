# Copilot Testing Kit 🚀

Welcome to the **Copilot Testing Kit** — a software testing toolkit for **GitHub Copilot** (VS Code agent mode and the GitHub Copilot CLI).

This repository provides ready-to-use working rules (Rules), reusable skills (Skills), in-depth workflows (Plans), and prompt templates so Copilot can effectively support both **Manual Testing** and **Automation Testing**.

The kit covers the full testing lifecycle: requirement analysis, test case design, test data generation, framework scaffolding, automation scripting, API testing, flaky-test handling, and Jira/Xray integration.

---

## 🌟 Key Features

- **🔁 End-to-end AI workflow:** Analyze requirements → design test cases → generate test data → write automation scripts → run tests → analyze and fix failures.
- **📋 Manual & automation testing:** Supports AI-RBT, test plans, traceability, Playwright, Selenium, Appium, and API testing.
- **🧠 26 specialized skills:** Copilot auto-selects a skill based on your request, or you can name a skill directly.
- **🧭 Consistent QA conventions:** POM, locator strategy, smart waits, deterministic data, and a delivery checklist.
- **🌐 Real UI exploration:** Inspect the DOM, generate stable locators, and avoid guessing selectors.
- **🛠️ Self-fix workflow:** Run tests, read failures, fix code, and re-verify with the appropriate test/lint/compile command.
- **🌍 English communication:** Guidance and reports are written in English.

---

## 📂 Main Directory Structure

```text
copilot-testing-kit/
├── .github/
│   └── copilot-instructions.md  # Repository-wide Copilot instructions
├── .agents/
│   ├── rules/                   # 6 QA and automation rule sets
│   └── skills/                  # 26 skills/workflows
├── plans/
│   ├── manual/                  # 6-step AI-RBT manual testing workflow
│   ├── automation/              # 6-step automation testing workflow
│   └── cross-module/            # Cross-module and combinatorial matrix
├── prompt_templates/            # Quick-use prompt templates
├── AGENTS.md                    # Detailed repository-level guidance
└── README.md
```

### `AGENTS.md` and `.github/copilot-instructions.md` — Repository Guidance

GitHub Copilot reads both `AGENTS.md` and `.github/copilot-instructions.md` before working in the repository (in VS Code agent mode and the Copilot CLI). `.github/copilot-instructions.md` is a concise entry point that points to `AGENTS.md`, which is the detailed source and defines:

- When to use the repository skills in `.agents/skills`.
- Which rules to read for Playwright, Selenium, Appium, locators, and test data.
- How to map plain-language capability descriptions ("read the file", "run in the terminal", "fetch the webpage") to the tools available in the current surface.
- Communication conventions, Git-state preservation, and how to validate automation code.

If a subproject adds its own `AGENTS.md` in a subdirectory, guidance closer to the working directory takes precedence.

### `.agents/skills/` — Skills and Workflows

Each skill is a folder containing a `SKILL.md` with `name`, `description`, execution steps, and related resources. Copilot auto-discovers skills by their `description` in both VS Code and the Copilot CLI. You can activate a skill in two ways:

1. **Let Copilot choose:** Describe your goal in natural language; Copilot matches it against each skill's `description`.
2. **Name it directly:** Mention the skill by its kebab-case name to steer Copilot to a specific workflow.

Key workflows:

| Skill                                  | Purpose                                                    |
| -------------------------------------- | ---------------------------------------------------------- |
| `generate-requirements-from-website`   | Analyze a website and generate requirements                |
| `analyze-requirement-document`         | Analyze a Jira ticket, user story, or requirement document |
| `generate-testcases-from-requirements` | Generate test cases quickly — QUICK mode                   |
| `generate-manual-testcases-rbt`        | Generate test cases with the full AI-RBT process           |
| `generate-application-test-plan`       | Explore an application and generate a test plan            |
| `generate-automation-framework`        | Scaffold a Playwright, Selenium, or Appium framework       |
| `generate-automation-from-testcases`   | Convert manual test cases into automation scripts          |
| `generate-automation-from-ui-flow`     | Generate automation from a UI flow and the real DOM        |
| `generate-api-tests-from-swagger`      | Generate API tests from Swagger/OpenAPI                    |
| `generate-locator`                     | Generate stable locators for UI elements                   |
| `generate-test-data`                   | Generate structured, traceable test data                   |
| `analyze-flaky-tests`                  | Analyze or fix flaky tests                                 |
| `generate-cross-module-test-plan`      | Generate a module map and combinatorial matrix             |
| `generate-combinatorial-test-data`     | Generate data for a cross-module matrix                    |
| `fetch-jira-requirements`              | Fetch requirements/user stories from Jira                  |
| `import-test-results-xray`             | Push automation results to Xray                            |

In addition to the workflows above, the repository includes foundational skills such as `qa-automation-engineer`, `rbt-manual-testing`, `requirements-analyzer`, `framework-architect`, `ui-debug-agent`, `smart-locator-agent`, `locator-healer-agent`, `test-data-generator`, `flaky-test-analyzer`, and `jira-integration`.

### `.agents/rules/` — Mandatory Rules

| File                    | Scope                                              |
| ----------------------- | -------------------------------------------------- |
| `automation_rules.md`   | Automation conventions, POM, naming, and test data |
| `locator_strategy.md`   | Priority order for stable locators                 |
| `playwright_rules.md`   | Playwright-specific rules                          |
| `selenium_rules.md`     | Selenium-specific rules                            |
| `appium_rules.md`       | Appium-specific rules                              |
| `delivery_checklist.md` | Verification checklist before delivery             |

These rules are not slash commands. `AGENTS.md` and each skill direct Copilot to read the rule files relevant to the current task.

---

## 🗺️ `plans/` — In-Depth Workflows

Use `plans/` when a task is complex and needs to be executed sequentially within a single conversation.

| Plan                  | Description                                                       | Quick start                                                              |
| --------------------- | ----------------------------------------------------------------- | ------------------------------------------------------------------------ |
| `plans/manual/`       | Generate manual test cases with the **6-step AI-RBT** process     | [`plans/manual/QUICK_START.md`](plans/manual/QUICK_START.md)             |
| `plans/automation/`   | Generate automation scripts with the context → review workflow    | [`plans/automation/QUICK_START.md`](plans/automation/QUICK_START.md)     |
| `plans/cross-module/` | Analyze multiple modules and generate a Pairwise/Cartesian matrix | [`plans/cross-module/QUICK_START.md`](plans/cross-module/QUICK_START.md) |

**How to use:** Open the appropriate `QUICK_START.md`, provide the input, and work through the checkpoints in order. Name the matching skill to steer Copilot to a specific workflow.

---

## 📝 `prompt_templates/` — Quick-Use Prompts

For single tasks: open a prompt → replace the content in `[...]` → paste it into Copilot Chat (VS Code) or the Copilot CLI → send.

| #   | Prompt                                       | Skill                                  |
| --- | -------------------------------------------- | -------------------------------------- |
| 01  | `prompt_01_generate_requirements.txt`        | `generate-requirements-from-website`   |
| 02  | `prompt_02_generate_test_cases.txt`          | `generate-testcases-from-requirements` |
| 03  | `prompt_03_create_framework_playwright.txt`  | `generate-automation-framework`        |
| 03  | `prompt_03_create_framework_selenium.txt`    | `generate-automation-framework`        |
| 04  | `prompt_04_generate_script_playwright.txt`   | `generate-automation-from-testcases`   |
| 04  | `prompt_04_generate_script_selenium.txt`     | `generate-automation-from-testcases`   |
| 05  | `prompt_05_convert_manual_to_automation.txt` | `generate-automation-from-testcases`   |
| 07  | `prompt_07_generate_test_data.txt`           | `generate-test-data`                   |
| 08  | `prompt_08_analyze_flaky_tests.txt`          | `analyze-flaky-tests`                  |
| 09  | `prompt_09_generate_api_tests.txt`           | `generate-api-tests-from-swagger`      |

> Prompt 02 uses **QUICK mode** via `generate-testcases-from-requirements` by default. For the full 6-step AI-RBT process, use `generate-manual-testcases-rbt` instead.

---

## ✳️ Using the Kit with GitHub Copilot

### Option 1: VS Code (Agent Mode)

1. Clone this repository, or copy `.github/`, `.agents/`, `AGENTS.md`, `plans/`, and `prompt_templates/` into the root of the project you want to test.
2. Open the project folder in VS Code with the GitHub Copilot and Copilot Chat extensions installed.
3. Open Copilot Chat and switch to **Agent** mode.
4. Describe your goal, or name a skill directly. For example:

   ```text
   Use generate-manual-testcases-rbt to create test cases for the requirement in requirements/login.md.
   ```

Copilot reads `AGENTS.md` and `.github/copilot-instructions.md`, auto-discovers the relevant skill, loads its `SKILL.md`, and runs the workflow.

### Option 2: GitHub Copilot CLI

1. Install the CLI (Node.js 22+):

   ```bash
   npm install -g @github/copilot
   ```

2. Open a terminal in the project directory that contains `.agents/` and `AGENTS.md`, then start Copilot:

   ```bash
   copilot
   ```

3. Authenticate with `/login` if prompted, and confirm you trust the directory.
4. Send a request and name the skill you want. For example:

   ```text
   Use generate-automation-from-testcases to convert the test cases in testcases/login.md into Playwright TypeScript.
   ```

The CLI auto-loads project skills from `.agents/skills` and reads `AGENTS.md` plus `.github/copilot-instructions.md`.

### Option 3: Let Copilot Choose the Skill

You do not need to remember skill names. Describe your goal in natural language:

```text
Analyze this Swagger file, generate API test cases, and create automation scripts using Playwright API.
```

Copilot matches your request against each skill's `description` and selects the right workflow. When you need precise control, name the skill directly.

---

## 🌐 Browser-Driven Skills (Playwright MCP)

Some skills (for example `ui-debug-agent`, `generate-locator`, and `generate-automation-from-ui-flow`) inspect a live page using `browser_*` steps. These require the **Playwright MCP server**:

- **VS Code:** Add the Playwright MCP server in your MCP configuration.
- **Copilot CLI:** Configure it via `/mcp`.

If the Playwright MCP server is not available, Copilot will tell you how to enable it instead of guessing selectors.

---

## 📚 Official GitHub Copilot Documentation

- [About GitHub Copilot CLI](https://docs.github.com/en/copilot/concepts/agents/about-copilot-cli)
- [Installing GitHub Copilot CLI](https://docs.github.com/en/copilot/how-tos/set-up/install-copilot-cli)
- [About agent skills](https://docs.github.com/en/copilot/concepts/agents/about-agent-skills)
- [Custom instructions for GitHub Copilot CLI](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/add-custom-instructions)
- [Use MCP servers with Copilot](https://docs.github.com/en/copilot/how-tos/context/model-context-protocol)

---
