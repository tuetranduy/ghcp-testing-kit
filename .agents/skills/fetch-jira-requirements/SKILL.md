---
name: fetch-jira-requirements
description: Fetch requirements / user stories from a Jira ticket and save them as a markdown or JSON file.
---

# Workflow: Fetch Jira Requirements

> **MANDATORY SKILL:** You MUST load and carefully read the content of the **`jira-integration`** skill (at `.agents/skills/jira-integration/SKILL.md`) to learn how to use the scripts before starting.

This workflow fetches requirements, user stories, or issues from Jira and turns them into documentation for test automation.

## Steps:

1. **Check prerequisites:**
   - Read the `jira-integration` skill to understand the script structure.
   - Check that the `.env` file exists and is configured correctly.
   - Check that dependencies are installed (`scripts/integrations/node_modules/`).
   - If not installed, run: `cd scripts/integrations && npm install`

2. **Determine what to fetch:**
   - Ask the user to provide:
     - A specific **Issue key** (e.g. `PROJ-123`) → use `--issue`
     - **Project key + Issue type** (e.g. `PROJ`, `Story`) → use `--project --type`
     - A custom **JQL query** → use `--jql`
     - An **Epic key** (to fetch children) → use `--epic`
   - Determine the output format: `json` (default) or `md` (markdown requirement)

3. **Run the script:**
   - Run the appropriate command:
   ```bash
   # Fetch a specific issue
   node scripts/integrations/jira/jira_fetcher.js --issue <ISSUE_KEY>

   # Fetch issues by project
   node scripts/integrations/jira/jira_fetcher.js --project <PROJECT_KEY> --type <TYPE> --max <N>

   # Search by JQL
   node scripts/integrations/jira/jira_fetcher.js --jql "<JQL_QUERY>"

   # Export Markdown
   node scripts/integrations/jira/jira_fetcher.js --issue <KEY> --format md --output ./requirements/jira
   ```

4. **Process the result:**
   - Check the output file created in `requirements/jira/` (or the path specified by `--output`).
   - If the format is `json`: Read and show a summary of the issues to the user.
   - If the format is `md`: Show the markdown requirement content for user review.
   - If an error occurs: Read the log and analyze the cause using the Troubleshooting table in the skill.

5. **Handle common errors:**
   - **HTTP 401**: Invalid token → guide the user to check `JIRA_API_TOKEN` or `JIRA_PAT`
   - **HTTP 404**: Issue does not exist or `JIRA_BASE_URL` is wrong
   - **File .env not found**: Guide the user to copy `.env.example` → `.env`
   - **Module not found**: Run `npm install` in `scripts/integrations/`

6. **Delivery:**
   - Present the results in English.
   - If the user asks to convert the output into a requirement document, use the `requirements-analyzer` skill to reformat it.
   - Save the output file to the appropriate folder (`requirements/jira/`).
