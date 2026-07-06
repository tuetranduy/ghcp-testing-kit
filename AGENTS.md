# GitHub Copilot workspace guidance

These instructions apply to GitHub Copilot in both VS Code (agent mode) and the GitHub Copilot CLI.

## QA automation resources

- Use repository skills from `.agents/skills` when the request matches a skill `description`, or when the user names a skill directly. Copilot auto-discovers these skills by their frontmatter `description` in both VS Code and the Copilot CLI.
- Read only the relevant rule files before implementing or reviewing automation code:
  - `.agents/rules/automation_rules.md` for shared automation and test-data conventions.
  - `.agents/rules/locator_strategy.md` for locator selection.
  - `.agents/rules/playwright_rules.md` for Playwright work.
  - `.agents/rules/selenium_rules.md` for Selenium work.
  - `.agents/rules/appium_rules.md` for Appium work.
  - `.agents/rules/delivery_checklist.md` before completing automation work.
- Refer to a skill by its kebab-case name, for example `generate-locator` or `analyze-flaky-tests`. There is no special invocation prefix; describe the goal in natural language and Copilot selects the matching skill.

## Tool usage

- Use the tools available in the current surface (VS Code agent tools or Copilot CLI tools): read files, create and edit files, run commands in the terminal, and fetch web content.
- Skill instructions describe capabilities in plain language ("read the file", "edit the file", "run in the terminal", "fetch the webpage"); map each to the equivalent tool available in the current session.
- Browser-driven steps (`browser_navigate`, `browser_snapshot`, `browser_click`, etc.) require the Playwright MCP server. Enable it in VS Code (MCP configuration) or in the Copilot CLI (`/mcp`). If the Playwright MCP server is unavailable, tell the user how to enable it instead of inventing tool calls.

## Working expectations

- Communicate and report results in concise, clear English.
- Preserve the current local code state. Do not run state-changing Git commands such as `git pull`, `git checkout`, `git merge`, `git rebase`, or `git reset` unless the user explicitly requests them. Read-only Git inspection is allowed.
- Prefer stable semantic locators and smart waits; do not introduce fixed sleeps unless the user explicitly requires them.
- Keep test data unique, traceable, deterministic when seeded, and free of real personal information.
- Validate generated automation with the narrowest relevant test, lint, or compile command available in the target project.
