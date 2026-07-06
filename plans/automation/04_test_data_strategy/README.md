# Step 4: Test Data Strategy

**Workflow:** generate-automation-from-testcases (continued)
**Skill:** qa-automation-engineer + test-data-generator

---

## Purpose

Test automation often becomes flaky when it uses hardcoded data. This step asks the AI to design a **random data generator** (Faker/Random) that is traceable, so the tests can run repeatedly / in parallel without collisions.

## How to use

1. Send the `prompt.txt` file to the AI.
2. The AI will generate:
   - A Utils/DataProvider class containing data-generation functions
   - Guidance for integrating it into the Test Cases
3. Review and integrate it into the project → move to Step 5.

## Standard Data Format

```
[Prefix]_[TestName]_[Timestamp]_[Random]
```

Example: `auto_createCustomer_20260402_A3F2@test.com`

→ From the database you can immediately tell: this data was produced by the `createCustomer` test on `2026-04-02`.

## Notes

- Each test method has its **own** data → parallel runs are safe.
- Use a Faker library for realistic data (person names, addresses, phone numbers, etc.).
- Follow the conventions in `.agents/rules/automation_rules.md` (Section 2: Test Data).
