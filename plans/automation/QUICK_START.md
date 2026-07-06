# 📋 Quick Start: 6-Step Automation Testing

## Two ways to use this

### Approach 1: One-Click (Fully automated — Recommended)

```
Use the generate-automation-from-testcases skill.

URL: [https://your-app.com/login]
Account: [admin@test.com / Test@123]
Framework: [Playwright TypeScript / Selenium Java]

Manual Test Cases:
[Paste test cases here]
```

→ The agent runs all 6 steps and auto-fixes until the tests PASS.

---

### Approach 2: Sequential (Step by step)

| Step  | Prompt to send                              | Wait for user?          |
| ----- | ------------------------------------------- | ----------------------- |
| **0** | See `project_architecture/README.md`        | One-time setup          |
| **1** | Copy `01.../prompt.txt` + fill in `[...]`   | ✅ Wait for confirmation |
| **2** | Copy `02.../prompt.txt` + fill in URL & TCs | ✅ Review locators       |
| **3** | Copy `03.../prompt.txt`                     | Review POM              |
| **4** | Copy `04.../prompt.txt`                     | Quick review            |
| **5** | Copy `05.../prompt.txt`                     | Wait for tests to PASS  |
| **6** | Copy `06.../prompt.txt`                     | Receive clean code      |

### Execution flow:

```
[Step 1] Establish role + tech stack
    ↓  AI confirms → OK
[Step 2] Provide URL + Test Cases
    ↓  AI opens the browser itself and collects locators
    ↓  ⏸️ User reviews the locator table
[Step 3] AI designs the POM classes
    ↓  User reviews the architecture
[Step 4] AI generates the Data Generator class
    ↓  User does a quick review
[Step 5] AI generates the test script + runs it
    ↓  Self-fix loop until it PASSes ✅
[Step 6] AI cleans up the code
    ↓  User receives clean code → commit ✅
```

---

## Optimization Tips

1. **Use Approach 1** for 80% of cases — it's the fastest and most effective.
2. **Use Approach 2** when the project is large or you need fine-grained control over each module.
3. **Always work within a single conversation** so the AI keeps its context.
4. **Provide a test account** if the app requires login.
