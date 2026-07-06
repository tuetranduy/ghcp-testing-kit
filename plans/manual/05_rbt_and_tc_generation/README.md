# Step 5: Detailed Test Case Generation (RBT & Test Case Generation)


---

## Purpose

Generate detailed Test Cases based on the **Risk-Based Testing (RBT)** strategy: high risk → test thoroughly, low risk → test the basics.

## How to use

1. Ensure the scenarios from Step 4 have been reviewed and confirmed.
2. Send the `prompt.txt` file to the AI, customizing the `[Hint]` sections if needed.
3. The AI will generate complete test cases: Title, Pre-condition, Steps, Expected Result, Test Data, Risk Level, Priority.
4. Review the results → move to Step 6.

## Important tips

### Split when there are many modules
If Step 4 has more than 5 modules, **don't ask the AI to generate them all at once**. Instead:
```
Generate Test Cases for Module 1 and Module 2 first.
```
After reviewing, continue:
```
Continue generating Test Cases for Module 3 and Module 4.
```

### Test Data must be specific
The AI should generate realistic mock test data instead of placeholders:
- ✅ `test_customer_01@domain.com` instead of ❌ `a valid email`
- ✅ `KH-2026-0012` instead of ❌ `a valid code`

### Risk Level determines depth
| Risk Level | Expected TC count | Coverage                           |
| ---------- | ----------------- | ---------------------------------- |
| **High**   | 8-15+ TC/module   | Happy + Negative + Boundary + Edge |
| **Medium** | 4-8 TC/module     | Happy + main Negative              |
| **Low**    | 2-4 TC/module     | Basic happy path                   |

### Test Case design techniques

The prompt integrates 4 classic techniques so the AI generates test cases systematically:

| Technique                    | When to use                             | Example                                         |
| ---------------------------- | --------------------------------------- | ----------------------------------------------- |
| **Equivalence Partitioning** | A field has many input types            | Age: <18, 18-60, >60                            |
| **Boundary Value Analysis**  | A field has min/max                     | Password 6-20 chars: test 5, 6, 20, 21          |
| **Decision Table**           | Logic with multiple combined conditions | Login: email + password + active status         |
| **State Transition**         | An object with a workflow/states        | Order: New → Processing → Delivered → Completed |
