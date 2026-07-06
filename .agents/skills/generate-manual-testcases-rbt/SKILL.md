---
name: generate-manual-testcases-rbt
description: Generate high-quality manual test cases following the 6-step AI-RBT (Risk-Based Testing) process from requirements.
---

> **MANDATORY SKILL:** You MUST load and carefully read the skill **`rbt-manual-testing`** (at `.agents/skills/rbt-manual-testing/SKILL.md`) before starting this task. Use the skill's **FULL RBT Mode**. Also refer to the skill **`requirements-analyzer`** to understand how to analyze the UI if needed.

# Workflow: Generate Manual Test Cases with the AI-RBT Framework (FULL RBT Mode)

This workflow uses the **FULL RBT Mode** of the `rbt-manual-testing` skill — the **AI-RBT (AI-Driven Risk-Based Testing)** process consisting of 6 sequential steps to generate manual test cases from a requirements document.

> [!NOTE]
> **This is a Copilot skill.** Follow the instructions in the skill; there is no need to read a `prompt.txt` file.
> If the QA team wants to use more detailed prompts, copy-paste them in sequence from `plans/manual/01_context_and_roleplay/prompt.txt` through `plans/manual/06_template_mapping/prompt.txt`.

## ⚠️ Execution principles

- **Mode:** FULL RBT (6 sequential steps)
- **You MUST run sequentially** step by step, do NOT combine multiple steps
- **You MUST stop** to wait for the user's response at Step 2 (Q&A) and Step 4 (Review Scenarios)
- If the user has not provided requirements, ask the user to provide them before starting
- All output in **English**

## Steps

Follow the detailed instructions in the `rbt-manual-testing` skill → the **Mode 2: FULL RBT** section.

### Step 1: Initialize the context (Context & Role-play)
1. Ask the user to provide: project name, system description, MVP goal, requirements document
2. Read the document carefully, confirm your understanding of the context
3. **Wait for user confirmation** → move to Step 2

### Step 2: Requirements analysis (Analysis & QnA)
1. Identify the Happy Path, Alternate Paths, Exception Paths
2. Detect Ambiguities (omissions, contradictions, unclear points)
3. Ask numbered Q&A questions (Q1, Q2...) to the user/PO/BA, with context + assumptions
4. **STOP — Wait for the user to answer the questions** → move to Step 3

### Step 3: Decompose the system (Decomposition)
1. Split the feature into Modules / Sub-modules
2. Describe each Module's functionality + the Dependencies between them

### Step 4: Ensure coverage (Traceability)
1. Map Module → requirement ID (REQ-01, REQ-02...)
2. Cross-check for omissions (Gap Analysis), list High-Level Scenarios
3. **Wait for the user to review** the scenarios → move to Step 5

### Step 5: Generate detailed Test Cases (RBT & TC Generation)
1. Assess the Risk Level (High/Medium/Low) for each Module
2. Generate complete test cases: Title, Pre-condition, Steps, Expected, Test Data, Priority
3. Apply techniques: EP, BVA, Decision Table, State Transition
4. **Field-Level Validation:**
   - List all input fields on the form/UI under test
   - Generate validation TCs **separately for EACH field** according to its own characteristics
   - Reference the **Field-Level Validation Table** in the `rbt-manual-testing` skill
   - Do **NOT** combine validation of multiple fields into 1 TC
5. Fully cover: Happy Path, Negative, Boundary, Edge Cases
6. Test Data must be specific (no generic placeholders)
7. If there is too much → generate module by module, ask the user to continue

### Step 6: Standardize the Format (Template Mapping)
1. Package all test cases into a standard Markdown table:
   `| TC ID | Module | Risk Level | Test Title | Pre-Condition | Test Steps | Expected Result | Priority | Test Data |`
2. Do not omit any test case
3. Export as an Artifact if it is long

## Output

- A complete Markdown Test Cases table, ready to copy into Excel/Jira/TestRail
- Traceability Matrix
- A list of resolved Ambiguities
