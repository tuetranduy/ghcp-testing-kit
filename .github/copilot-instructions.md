# Copilot Testing Kit — repository instructions

This repository is a QA testing kit for GitHub Copilot (VS Code agent mode and the Copilot CLI). It provides reusable skills, QA rules, step-by-step plans, and prompt templates for both manual and automation testing.

- Skills live in `.agents/skills/<skill-name>/SKILL.md` and are auto-discovered by their `description`. Refer to a skill by its kebab-case name; describe the goal in natural language rather than using any special prefix.
- Mandatory QA conventions live in `.agents/rules/`. Read only the rule files relevant to the current task.
- `AGENTS.md` at the repository root is the detailed source of guidance for tool usage, browser/MCP setup, and working expectations. Follow it.

Communicate in English. Preserve the local Git state unless the user explicitly asks otherwise.
