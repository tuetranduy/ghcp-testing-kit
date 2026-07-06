---
name: generate-testcases-from-requirements
description: Quickly generate manual test cases from requirements (QUICK mode — without the 6-step process).
---

> **MANDATORY SKILL:** You MUST load and carefully read the content of the **`rbt-manual-testing`** skill (at `.agents/skills/rbt-manual-testing/SKILL.md`) before starting this task. Use the **QUICK Mode** of that skill.

# Workflow: Quickly Generate Manual Test Cases from Requirements

This workflow uses the **QUICK Mode** of the `rbt-manual-testing` skill to quickly generate test cases from existing requirements.

## ⚠️ Principles

- **Mode:** QUICK (a single pass, no waiting for the user midway)
- Suitable for simple modules with already-clear requirements
- If the requirements turn out to be too complex or ambiguous → **automatically switch to FULL RBT** and notify the user
- All output in **English**

## Steps

1. **Read and understand the requirements** provided by the user
2. **Identify the main flows:** Happy Path, Negative Path, Boundary Cases, Edge Cases
3. **Apply automated test case design techniques:**
   - Equivalence Partitioning (EP)
   - Boundary Value Analysis (BVA)
   - Decision Table (if there are many rules)
   - State Transition (if there is a workflow)
4. **Field-Level Validation:**
   - List all input fields on the form/UI
   - Generate validation TCs **for EACH field separately** according to its own characteristics (text, email, phone, date, number, dropdown, file upload, password...)
   - Apply the **Field-Level Validation table** in the `rbt-manual-testing` skill to choose appropriate validations
   - Do **NOT** combine validation of multiple fields into one test case
5. **Generate test cases with complete fields:**
   - TC ID (format: `[PROJECT]_[MODULE]_TC_[NUMBER]`)
   - Module
   - Test Scenario / Test Case Title
   - Pre-conditions
   - Test Steps (numbered)
   - Expected Results (correspondingly numbered)
   - Test Data (**must be concrete**, no placeholders)
   - Priority (Critical / High / Medium / Low)
6. **Export to a standard Markdown table**

## Output Table

```
| TC ID | Module | Test Scenario | Pre-Condition | Test Steps | Test Data | Expected Result | Priority |
```

## Important Rules

- Test Data must be concrete: `test_login_01@domain.com`, not "valid email"
- Must include Positive, Negative, Boundary, and Edge cases
- Each input field must have its own validation TCs (do not combine multiple fields into one TC)
- TC ID follows a consistent format defined by the user or the default `[PROJECT]_[MODULE]_TC_[NUMBER]`
- If there are too many TCs → split into Part 1, Part 2 and ask the user

## When to Switch to FULL RBT

The agent **automatically suggests switching mode** if it detects:
- Ambiguous requirements that need a Q&A
- Large scope (>3 modules)
- Complex business logic with many overlapping conditions
- The user requests a Traceability Matrix or Risk Assessment