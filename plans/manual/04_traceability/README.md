# Step 4: Traceability & Gap Analysis


---

## Purpose

Cross-check and establish a **Traceability Matrix** to ensure 100% of the original requirements are covered by Test Scenarios.

## How to use

1. Send the `prompt.txt` file after reviewing the decomposition results from Step 3.
2. The AI will return:
   - **Traceability Matrix:** A table mapping REQ ID ↔ Module ↔ Scenario
   - **Gap Analysis:** A report of any missing items
   - **High-Level Scenarios:** A list of high-level scenarios
3. **Review the scenarios carefully:**
   - Is any scenario missing?
   - Is any scenario redundant / duplicated?
   - Add more if needed → Confirm → move to Step 5.

## ⚠️ Human Checkpoint

**This is a human sign-off point.** Reasons:

- The AI may miss project-specific edge cases.
- The human tester needs to personally assess the **risk level** of each module **before** letting the AI generate detailed test cases in Step 5.
- If something is missing, ask the AI to add it: *"Add scenarios for the case [X]"*.
