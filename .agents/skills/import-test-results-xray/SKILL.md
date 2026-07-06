---
name: import-test-results-xray
description: Push test automation results (Playwright/JUnit/Allure) to Xray Test Management.
---

# Workflow: Import Test Results to Xray

> **MANDATORY SKILL:** You MUST load and carefully read the content of the **`jira-integration`** skill (at `.agents/skills/jira-integration/SKILL.md`) to learn how to use the scripts before starting.

This workflow helps push test automation results to Xray (Cloud or Server/Data Center) for tracking in Jira.

## Steps:

1. **Check prerequisites:**
   - Read the `jira-integration` skill to understand the script structure.
   - Check that the `.env` file has Xray credentials configured:
     - **Xray Cloud**: `XRAY_CLIENT_ID`, `XRAY_CLIENT_SECRET`
     - **Xray Server**: `JIRA_PAT` or `JIRA_EMAIL` + `JIRA_API_TOKEN`
   - Check `XRAY_PLATFORM` (cloud/server) in `.env`.
   - If dependencies are not installed: `cd scripts/integrations && npm install`

2. **Verify Xray authentication:**
   ```bash
   node scripts/integrations/jira/xray_auth.js --verify
   ```
   - If it fails: check the credentials in `.env`

3. **Determine the report to import:**
   - Ask the user which report type:
     - **Playwright JSON**: `--format playwright --file <path-to-results.json>`
     - **JUnit XML**: `--format junit --file <path-to-junit.xml>`
     - **Xray JSON** (already converted): `--format xray --file <path-to-xray.json>`
   - Confirm the **Project Key** (from `.env` or `--project`)

4. **Execute the import:**
   ```bash
   # Playwright report
   node scripts/integrations/jira/xray_importer.js --format playwright --file ./test-results.json --project PROJ

   # JUnit XML
   node scripts/integrations/jira/xray_importer.js --format junit --file ./junit-results.xml --project PROJ

   # Xray JSON
   node scripts/integrations/jira/xray_importer.js --format xray --file ./xray-payload.json
   ```

5. **Handle the result:**
   - Check the output: the Test Execution Key created in Jira.
   - On success: notify the user of the new Test Execution key.
   - On failure: read the log → analyze the cause → fix → run again.

6. **Important notes:**
   - **Test Key Convention**: The test title should contain the Jira key so Xray maps automatically:
     ```typescript
     test('[PROJ-123] Login should work', async ({ page }) => { ... });
     ```
   - **Xray Cloud** requires separate authentication (Client ID + Secret), different from Jira auth.
   - **Rate limit**: Avoid importing too many times in quick succession.
