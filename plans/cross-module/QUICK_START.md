# 📋 Quick Start: Cross-Module Testing & Combination Matrix

## 🔀 Choose Your Workflow

### Workflow 1: Agent skill — Automated (Recommended ⭐)

> Use this when working with an AI agent that supports skills.

```
Use the generate-cross-module-test-plan skill.

Feature: [Feature name, e.g. "Partner payment record"]
URL: [https://your-app.com]
Account: [admin@test.com / Test@123]

Related modules:
1. [Module 1: Partner management — choose partner type]
2. [Module 2: Create payment — choose VND/USD]
3. [Module 3: Tax configuration — choose tax type]
4. [Module 4: Debt management — choose debt type]
5. [Final module: Generate record — output]

Matrix strategy: pairwise (or: business-critical / full-cartesian)
```

→ The AI opens the browser → explores each module → draws the Data Flow → generates the combination matrix.

---

### Workflow 2: Copy-Paste into any AI chat — Manual

> Use this when you want to use a different AI.

**Sample prompt — Copy and paste into the AI chat:**

```
You are a Senior QA Engineer specializing in Combinatorial Testing.

I have a feature "[FEATURE NAME]" that traverses multiple sequential modules.
Each module has multiple choices, and the combination of choices determines the final output.

Modules and dimensions:
- Module 1 [NAME]: dimension [DIMENSION NAME] = [value 1, value 2, ...]
- Module 2 [NAME]: dimension [DIMENSION NAME] = [value 1, value 2, ...]
- Module 3 [NAME]: dimension [DIMENSION NAME] = [value 1, value 2, ...]
- ...

Please:
1. Draw a Data Flow Diagram between the modules (which module outputs what → input to which module)
2. List all constraints (invalid combinations)
3. Generate a combination matrix table using the Pairwise Testing strategy
4. For each combination, state the Expected Output clearly (template, formula if any)

Output as a Markdown table, ready to copy into Excel.
```

---

## 🎯 End-to-End Flow — Step by Step

### Step 1: Cross-Module Analysis & Matrix Generation

```
Use the generate-cross-module-test-plan skill.

Feature: Partner payment record
URL: https://example.com/partners
```

**Result:** The AI generates:
- 📊 A Data Flow Diagram (which module → which module)
- 📋 A Dimensions table (all dimensions + values)
- 📈 **The combination matrix** (Pairwise ~20 combos instead of 216 Full combos)

**⏸️ User review:** Check the matrix → add/edit → confirm OK.

---

### Step 2: Generate Test Data for the Matrix

```
Use the generate-combinatorial-test-data skill.

Matrix: [paste the matrix table from Step 1]
Mode: GENERATE (or PIPELINE if you want to create real data on the system)
Format: json (or: csv, markdown, typescript)
```

**Result:**
- GENERATE mode: A JSON/CSV file containing N data sets, each set = 1 combo
- PIPELINE mode: The AI runs a real browser → creates data on the system → reports pass/fail

---

### Step 3: Generate Test Cases (Optional)

```
Use the generate-manual-testcases-rbt skill.

Requirements: [paste requirements + the combination matrix from Step 1]
```

→ The AI generates detailed test cases for each important combination.

---

### Step 4: Generate Automation Scripts (Optional)

```
Use the generate-automation-from-testcases skill.

URL: https://example.com
Test cases: [paste the test cases from Step 3]
Framework: Playwright TypeScript
```

→ The AI generates scripts + runs them + fixes them → stable PASS.

---

## 📊 Real-World Example: Partner Payment Record

### Input (you provide):

```
Feature: Partner payment record
Modules:
1. Partner management: Type = [Organization, Individual, Business household]
2. Payment: Type = [VND, USD]
3. Tax: Type = [PIT, VAT, Contractor, Tax-exempt]
4. Debt: Type = [Regular, Advance, Adjustment]
5. Asset source: Type = [Fund A, Fund B, Fund C]
```

### Output (the AI generates):

**Data Flow:**
```
Partner → Payment → Tax → Debt → Record
(type)     (currency)  (Depends on   (Depends on
                        type+currency)  everything)
```

**Pairwise Matrix (20 combos instead of 3×2×4×3×3 = 216 combos):**

| #   | Partner            | Pay | Tax        | Debt       | Source | Expected        |
| --- | ------------------ | --- | ---------- | ---------- | ------ | --------------- |
| 1   | Organization       | VND | VAT        | Regular    | Fund A | REC_ORG_VND_VAT |
| 2   | Organization       | USD | PIT        | Advance    | Fund B | REC_ORG_USD_PIT |
| 3   | Individual         | VND | PIT        | Regular    | Fund A | REC_IND_VND_PIT |
| 4   | Individual         | USD | VAT        | Adjustment | Fund C | REC_IND_USD_VAT |
| 5   | Business household | VND | Contractor | Regular    | Fund B | REC_BH_VND_CT   |
| ... | ...                | ... | ...        | ...        | ...    | ...             |

→ Cover 100% of pairs between any 2 dimensions with only ~20 combos!

---

## 💡 Optimization Tips

1. **Start with Pairwise** — good enough for 90% of cases, reduces effort by 80-90%
2. **Provide business rules** if available — the AI will map them into the "Expected Output" column more accurately
3. **Review the Data Flow at Step 2** — this is the most important checkpoint; an error here → the matrix will be wrong
4. **Use PIPELINE mode** when test data must be created for real through the UI (cannot seed the database)
5. **Run within the same conversation** with an agent skill so the AI retains context throughout
6. **Split into batches** if the matrix has more than 30 combos — avoid timeouts and maintain quality

---

## ⚠️ Distinction: When Do You NOT Need This Workflow?

| Situation                                                              | Which workflow to use?                                                    |
| ---------------------------------------------------------------------- | ------------------------------------------------------------------------- |
| Test a single module, simple form                                      | `generate-manual-testcases-rbt` or `generate-testcases-from-requirements` |
| Multiple **independent** modules (no effect on each other)             | `generate-application-test-plan`                                          |
| Multiple **sequential** modules, output depends on the **combination** | ✅ `generate-cross-module-test-plan` ← **Use this workflow**               |
| Only need test data for one form                                       | `generate-test-data`                                                      |
