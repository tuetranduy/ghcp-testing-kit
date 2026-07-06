# Step 3: Design the POM Structure (POM Design)

**Workflow:** generate-automation-from-testcases (continued)
**Skill:** qa-automation-engineer

---

## Purpose

After the AI has collected locators from the real DOM in Step 2, this step guides the AI to shape the **Page classes** following the POM standard. The goal is to allocate the right files and give methods clear, meaningful names **before** implementing the test logic.

## How to use

1. Make sure you have reviewed and confirmed the locators from Step 2.
2. Send the `prompt.txt` file to the AI.
3. The AI will outline:
   - Page class names + file paths
   - Locator declarations (from Step 2)
   - Method signatures + bodies
4. Review the POM architecture → move to Step 4.

## Naming Conventions

| Component      | Rule                                     | Example                                   |
| -------------- | ---------------------------------------- | ----------------------------------------- |
| **Class name** | PascalCase + `Page` suffix               | `LoginPage`, `CustomerFormPage`           |
| **Locator**    | camelCase, describes the element         | `emailInput`, `submitButton`              |
| **Method**     | camelCase, describes the business action | `fillLoginForm()`, `verifySuccessToast()` |
| **File**       | Per the framework convention             | `LoginPage.ts` / `LoginPage.java`         |

## Notes

- This step only designs the **Page classes**; it does NOT yet generate the test script.
- Methods should return `this` or the target Page (fluent pattern) so they can be chained.
- Follow the conventions in `.agents/rules/automation_rules.md`.
