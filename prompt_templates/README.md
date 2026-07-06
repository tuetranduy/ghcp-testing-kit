# Copilot Prompt Templates

This folder contains ready-to-use prompt templates that reference a **skill** on their first line.

When you paste a prompt into Copilot Chat (VS Code) or the Copilot CLI, the first line names the skill in natural language. Copilot auto-discovers skills by their `description`, reads the matching `SKILL.md` in `.agents/skills/`, and runs the correct workflow.

---

## Prompt List

| #   | File                                         | Workflow skill                         | Foundation skill         |
| --- | -------------------------------------------- | -------------------------------------- | ------------------------ |
| 01  | `prompt_01_generate_requirements.txt`        | `generate-requirements-from-website`   | `requirements-analyzer`  |
| 02  | `prompt_02_generate_test_cases.txt`          | `generate-testcases-from-requirements` | `rbt-manual-testing`     |
| 03  | `prompt_03_create_framework_playwright.txt`  | `generate-automation-framework`        | `framework-architect`    |
| 03  | `prompt_03_create_framework_selenium.txt`    | `generate-automation-framework`        | `framework-architect`    |
| 04  | `prompt_04_generate_script_playwright.txt`   | `generate-automation-from-testcases`   | `qa-automation-engineer` |
| 04  | `prompt_04_generate_script_selenium.txt`     | `generate-automation-from-testcases`   | `qa-automation-engineer` |
| 05  | `prompt_05_convert_manual_to_automation.txt` | `generate-automation-from-testcases`   | `qa-automation-engineer` |
| 07  | `prompt_07_generate_test_data.txt`           | `generate-test-data`                   | `test-data-generator`    |
| 08  | `prompt_08_analyze_flaky_tests.txt`          | `analyze-flaky-tests`                  | `flaky-test-analyzer`    |
| 09  | `prompt_09_generate_api_tests.txt`           | `generate-api-tests-from-swagger`      | `qa-automation-engineer` |

> Prompt 02 defaults to **QUICK mode** via `generate-testcases-from-requirements`. To run the full 6-step AI-RBT process, change the first line to reference the `generate-manual-testcases-rbt` skill.

## How to Use

1. Choose the appropriate prompt file.
2. Replace the `[...]` placeholders with real data.
3. Keep the first line that names the skill.
4. Copy the entire content and paste it into Copilot Chat (VS Code) or the Copilot CLI.
5. Follow the checkpoints or clarifying questions requested by the skill.

## Naming Conventions

- The first line of each prompt names the skill in natural language; Copilot auto-discovers skills by their `description`.
- Skill names use **kebab-case**, for example `generate-automation-from-testcases`.
- The physical skill files live at `.agents/skills/<skill-name>/SKILL.md`.
- Do not use slash commands.
