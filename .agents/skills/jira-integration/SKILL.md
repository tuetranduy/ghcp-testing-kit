---
name: jira-integration
description: Jira/Xray integration skill — fetch requirements from Jira, authenticate with Xray, and push test results to Xray Cloud/Server.
---

# Jira & Xray Integration Skill

## Description

This skill provides integration between the GitHub Copilot workspace and Jira/Xray systems to:

1. **Fetch Requirements/User Stories** from Jira → convert them into a standard requirements document
2. **Authenticate with Xray** (Cloud or Server/Data Center)
3. **Push test results** (Playwright, JUnit, Allure) to Xray Test Management

---

## When to Use

The agent uses this skill when the user asks to:

- Fetch a requirement / user story from Jira
- Connect / test a connection to the Jira API
- Push test results to Xray
- Import test results into Jira
- Authenticate an Xray token
- Integrate CI/CD with Jira

Trigger keywords:
- "fetch jira", "get jira requirement", "get jira ticket"
- "import xray", "push results to xray", "push test results"
- "test jira connection", "check jira connection"

---

## Script Structure

```
scripts/integrations/
├── jira/
│   ├── jira_fetcher.js      # Fetch Requirement/User Story from Jira
│   ├── xray_auth.js         # Authenticate and obtain an Xray token
│   ├── xray_importer.js     # Import test results to Xray
│   └── utils.js             # Shared utility functions
└── package.json             # Dependencies (axios, dotenv)
```

---

## Prerequisites

### 1. Install dependencies

```bash
cd scripts/integrations
npm install
```

### 2. Configure .env

Copy `.env.example` to `.env` in the project root:

```bash
cp .env.example .env
```

Fill in the required information:

| Variable | Description | Required |
|----------|-------------|----------|
| `JIRA_BASE_URL` | Jira instance URL (e.g., `https://domain.atlassian.net`) | ✅ |
| `JIRA_EMAIL` | Jira account email (Cloud) | ✅ (Cloud) |
| `JIRA_API_TOKEN` | API Token (Cloud) | ✅ (Cloud) |
| `JIRA_PAT` | Personal Access Token (Server/DC) | ✅ (Server) |
| `JIRA_PROJECT_KEY` | Default project key | Recommended |
| `XRAY_PLATFORM` | `cloud` or `server` | Default: cloud |
| `XRAY_CLIENT_ID` | Xray API Client ID | When using Xray Cloud |
| `XRAY_CLIENT_SECRET` | Xray API Client Secret | When using Xray Cloud |

### 3. How to get a Jira API Token (Cloud)

1. Sign in at [https://id.atlassian.com/manage-profile/security/api-tokens](https://id.atlassian.com/manage-profile/security/api-tokens)
2. Click **Create API token**
3. Set a label (for example: "QA Automation")
4. Copy the token → paste it into `JIRA_API_TOKEN` in the `.env` file

### 4. How to get an Xray Cloud API Key

1. Go to [https://app.getxray.app](https://app.getxray.app) → Settings → API Keys
2. Or in Jira: Apps → Xray → Settings → API Keys
3. Create a new API Key → Copy the **Client ID** and **Client Secret**

---

## Usage Guide

### Fetch a specific issue

```bash
node scripts/integrations/jira/jira_fetcher.js --issue PROJ-123
```

### Fetch issues by project

```bash
node scripts/integrations/jira/jira_fetcher.js --project PROJ --type Story --max 20
```

### Search by JQL

```bash
node scripts/integrations/jira/jira_fetcher.js --jql "project = PROJ AND status = 'To Do'"
```

### Export as a Markdown requirement

```bash
node scripts/integrations/jira/jira_fetcher.js --issue PROJ-123 --format md
```

### Fetch the children of an Epic

```bash
node scripts/integrations/jira/jira_fetcher.js --epic PROJ-10 --format md
```

### Test Xray authentication

```bash
node scripts/integrations/jira/xray_auth.js
node scripts/integrations/jira/xray_auth.js --verify
```

### Import Playwright results to Xray

```bash
node scripts/integrations/jira/xray_importer.js --format playwright --file ./test-results.json --project PROJ
```

### Import JUnit XML to Xray

```bash
node scripts/integrations/jira/xray_importer.js --format junit --file ./junit-results.xml --project PROJ
```

---

## Related Workflows

| Workflow | Description |
|----------|-------------|
| `fetch-jira-requirements` | Fetch requirements from a Jira ticket and save them to a file |
| `import-test-results-xray` | Push test results to Xray |

---

## Important Notes

- **Security**: NEVER commit the `.env` file to Git. The `.gitignore` file is already configured to ignore `.env`.
- **Rate Limiting**: Jira Cloud has an API call limit. The script supports pagination to avoid exceeding the limit.
- **Atlassian Document Format (ADF)**: Jira Cloud uses ADF for descriptions. The script automatically converts ADF → plain text.
- **Test Key Convention**: When importing Playwright results, put the test key in the title: `test('[PROJ-123] Login should work', ...)` so Xray maps to the correct test case.

---

## Troubleshooting

| Error | Cause | Solution |
|-------|-------|----------|
| HTTP 401 | Wrong token/password | Check `JIRA_API_TOKEN` or `JIRA_PAT` |
| HTTP 403 | No permission | Check permissions on the Jira project |
| HTTP 404 | Wrong URL or the issue does not exist | Check `JIRA_BASE_URL` and the issue key |
| `ENOTFOUND` | DNS does not resolve | Check whether `JIRA_BASE_URL` has the correct domain |
| `ECONNREFUSED` | Server is not running | Check whether Jira Server is online |
| File .env not found | `.env` not created | Copy `.env.example` → `.env` |
