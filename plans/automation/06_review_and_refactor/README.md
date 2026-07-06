# Step 6: Review & Refactoring

**Workflow:** generate-automation-from-testcases (continued)
**Skill:** qa-automation-engineer

---

## Purpose

After the tests PASS, "clean up" the code to meet the **Clean Code** standard before merging into the repository. This avoids junk code polluting the project.

## How to use

1. Send the `prompt.txt` file after the tests have PASSed in Step 5.
2. The AI will:
   - Remove debug logs, commented code, and unused variables
   - Review the code quality
   - Add CI/CD tags (smoke, regression, etc.)
3. Receive the final code → commit/merge.

## Review Checklist

| #   | Check                                        | Status |
| --- | -------------------------------------------- | ------ |
| 1   | No more `console.log` / `System.out.println` | ☐      |
| 2   | No more commented code                       | ☐      |
| 3   | No unused variables/locators                 | ☐      |
| 4   | No hard sleeps                               | ☐      |
| 5   | Assertions are complete and correct          | ☐      |
| 6   | Test data is unique and traceable            | ☐      |
| 7   | Each test case is independent                | ☐      |
| 8   | CI/CD tags are added                         | ☐      |

## Notes

- This step can run automatically in the One-Click workflow (the AI cleans up after the tests PASS).
- If running manually, paste the source code in for the AI to review.
- The code must meet the `.agents/rules/automation_rules.md` standard before committing.
