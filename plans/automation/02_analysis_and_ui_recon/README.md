# Step 2: Analysis & UI Recon

**Workflow:** generate-automation-from-testcases (continued)
**Skill:** qa-automation-engineer + ui-debug-agent

---

## Purpose

Instead of a human manually inspecting the DOM, this step assigns the AI the task of **opening the browser itself**, navigating the web, and extracting real locators from the DOM. This is the step that separates "guesswork" automation from "real inspection" automation.

## How to use

1. Fill in the URL, test account, and Test Cases in the `prompt.txt` file.
2. Send it to the AI — the AI will use the Playwright/Selenium MCP to:
   - Launch the browser itself
   - Navigate through the test steps
   - Collect locators from the real DOM
3. Review the locator table the AI returns.
4. If any locator is incorrect → ask the AI to re-inspect that element.
5. Confirm → move to Step 3.

## Important Notes

- The AI uses the **Accessibility Tree** and **DOM inspection** to find locators — it does not guess.
- If login is required, provide a test account in the prompt.
- Default viewport is **1920x1080** (desktop) per the conventions in the rules.
- The AI prioritizes locators in the order defined in `.agents/rules/locator_strategy.md`.
