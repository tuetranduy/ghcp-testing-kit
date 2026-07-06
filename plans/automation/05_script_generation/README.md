# Step 5: Script Generation

**Workflow:** generate-automation-from-testcases (continued)
**Skill:** qa-automation-engineer

---

## Purpose

The core step — the AI combines the POM (Step 3) and the Data Strategy (Step 4) to generate a complete Test script, run the tests, and self-fix on failure.

## How to use

1. Send the `prompt.txt` file to the AI.
2. The AI will:
   - Generate a complete Test script file
   - **Run the tests itself** in the terminal
   - If they fail → **read the logs, fix them, and re-run itself**
   - Loop until they PASS reliably
3. When the tests PASS → move to Step 6.

## Code Pattern: Arrange → Act → Assert

```
Arrange: Set up data, initialize pages
Act:     Perform the action
Assert:  Check the result
```

## Self-fix Loop

The AI runs this loop:
```
Generate code → Run tests → FAIL?
    ├── Read the error log
    ├── Analyze the root cause
    ├── Fix the code
    └── Re-run → PASS? → DONE ✅
```

## Notes

- The AI will **not ask the user** during the self-fix process (unless it hits a conflicting business rule).
- Smart Waits are mandatory — see `.agents/rules/playwright_rules.md` or `.agents/rules/selenium_rules.md`.
- Every test case must have a **clear assertion** at the end.
