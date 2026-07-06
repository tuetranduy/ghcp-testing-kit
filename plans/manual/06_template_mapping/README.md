# Step 6: Format Standardization (Template Mapping)


---

## Purpose

Package all the Test Cases generated in Step 5 into a standard Markdown table, ready to copy into **Excel**, **Google Sheets**, or import directly into **Jira/TestRail/Xray/Zephyr**.

## How to use

1. Send the `prompt.txt` file after reviewing the test cases in Step 5.
2. The AI will output a Markdown table with the format:
   ```
   | TC ID | Module | Risk Level | Test Title | Pre-Condition | Test Steps | Expected Result | Priority | Test Data |
   ```
3. Copy the resulting table → paste it into your test management tool.

## TC ID rules

Default format: `[PROJECT]_[MODULE]_TC_[NUMBER]`

Example: `CRM_CUST_TC_001`, `CRM_LOGIN_TC_001`

If the project has its own ID convention, change it in the `[Customize]` section of prompt.txt.

## Handling when too long

If the total number of Test Cases exceeds 30, the AI will automatically split them into **Part 1, Part 2...** and ask you before continuing. Ensure:
- No TC is missed between parts
- TC ID numbering is continuous across parts
