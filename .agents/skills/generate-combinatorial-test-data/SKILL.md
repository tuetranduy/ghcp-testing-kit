---
name: generate-combinatorial-test-data
description: Generate test data for a multi-dimensional combination matrix through a live pipeline across multiple modules. Uses the output of the generate-cross-module-test-plan skill as input.
---

# generate-combinatorial-test-data — Generate Test Data for a Combination Matrix

> **Use when:** You already have a combination matrix (from `generate-cross-module-test-plan`) and need to **create real test data** by running through multiple modules in the browser, or generate a structured data set ready for automation.

> **MANDATORY:** Before starting, you MUST load and carefully read:
> - **Skill:** `.agents/skills/test-data-generator/SKILL.md` — Data generation rules (see the Multi-Step Pipeline section)
> - **Skill:** `.agents/skills/ui-debug-agent/SKILL.md` — Inspect the DOM when running the browser
> - **Workflow:** `.agents/skills/generate-cross-module-test-plan/SKILL.md` — Understand the input matrix structure

---

## Relationship with other workflows

```
generate-cross-module-test-plan     →  Combination matrix (input)
        ↓
generate-combinatorial-test-data    →  Test data set (this workflow)
        ↓
generate-manual-testcases-rbt       →  Detailed test cases
        ↓
generate-automation-from-testcases  →  Automation scripts
```

---

## 2 Modes

| Mode | When to use | Output |
|------|-------------|--------|
| **GENERATE** (default) | Generate a structured data set from the matrix — do NOT run the browser | JSON/CSV/Markdown file with data for each combination set |
| **PIPELINE** | Need to create REAL data in the system by running through each module in the browser | Real data created + IDs + screenshot evidence |

> The agent selects the mode automatically:
> - User says "generate data" → **GENERATE**
> - User says "create data in the system", "run to create data", "set up real data" → **PIPELINE**
> - If unclear → ask the user

---

## Inputs required from the User

| Input | Required | Description |
|-------|----------|-------|
| **Combination matrix** | ✅ | `.md` file / Markdown table from `generate-cross-module-test-plan` |
| **Application URL** | ✅ (PIPELINE) | So the agent can run the browser to create data |
| **Credentials** | ⚠️ PIPELINE | If the app requires login |
| **Output format** | ❌ | `json` (default), `csv`, `markdown`, `code` (TS/Java/Python) |
| **Data language** | ❌ | English / Vietnamese (default: based on context) |

---

## Steps

### Step 1: Read & Parse the Combination Matrix

1. **Read the matrix file** provided by the user:
   - Local file → read the file
   - Inline in chat → parse directly
   - URL → fetch the webpage

2. **Parse and validate:**
   - Identify the list of dimensions (D1, D2, D3...)
   - Identify the values of each dimension
   - Read the expected template/formula for each set
   - Count the total number of combination sets that need data

3. **Present a summary:**
   ```markdown
   📊 Matrix read:
   - Dimensions: 5 (Partner, Payment, Tax, Debt, Asset Source)
   - Total combination sets: 20 (Pairwise)
   - Mode: GENERATE / PIPELINE
   
   Start generating data? (Y/N)
   ```

---

### Step 2: Analyze Fields & Data Requirements per Module

1. **For each module** in the chain, identify the fields that need data:

   ```markdown
   | Module | Field | Type | Required | Constraints | Data Source |
   |--------|-------|------|----------|-------------|-------------|
   | Partner | partner_name | string | ✅ | max: 200 | Random + prefix |
   | Partner | partner_type | select | ✅ | enum: [TC, CN, HKD] | From dimension D1 |
   | Partner | tax_id | string | ✅ | 10-13 digits | Random unique |
   | Payment | currency | select | ✅ | enum: [VND, USD] | From dimension D2 |
   | Payment | amount | number | ✅ | min: 1 | Business-relevant values |
   | Tax | tax_type | select | ✅ | enum: [PIT, VAT, NT, MT] | From dimension D3 |
   | ...| ... | ... | ... | ... | ... |
   ```

2. **Classify the fields:**

   | Type | Description | How to generate data |
   |------|-------|----------------|
   | **Dimension fields** | A value belonging to a dimension in the matrix | Take it from the combination set (not random) |
   | **Supporting fields** | Required fields that are not dimensions | Generate random + unique + traceable |
   | **Computed fields** | Calculated from a formula | Compute per the business rules |
   | **Reference fields** | ID/code from a previous module | Copy from the previous module's output |

---

### Step 3: Generate Test Data — GENERATE Mode

> Perform when mode = GENERATE (default)

1. **For each combination set** in the matrix, generate 1 complete data set:

   ```json
   {
     "combination_id": "COMBO_01",
     "dimensions": {
       "D1_partner_type": "Organization",
       "D2_payment_type": "VND",
       "D3_tax_type": "VAT 10%",
       "D4_debt_type": "Standard",
       "D5_asset_source": "Fund A"
     },
     "module_data": {
       "module_1_partner": {
         "partner_name": "auto_combo01_tc_1712049200",
         "partner_type": "Organization",
         "tax_id": "0123456789",
         "address": "1 Nguyen Hue, District 1, HCMC"
       },
       "module_2_payment": {
         "currency": "VND",
         "amount": 100000000,
         "payment_date": "2026-04-15",
         "description": "Payment combo01"
       },
       "module_3_tax": {
         "tax_type": "VAT",
         "tax_rate": 10,
         "tax_amount": 10000000
       },
       "module_4_debt": {
         "debt_type": "Standard",
         "advance_amount": 0
       }
     },
     "expected_output": {
       "template": "BB_TC_VND_VAT",
       "formula": "Amount × 1.10",
       "computed_total": 110000000,
       "expected_fields": ["partner_name", "tax_id", "amount", "tax_amount", "total"]
     }
   }
   ```

2. **Generate enough data for ALL combination sets** → package into 1 output file

3. **Ensure data rules:**
   - Unique per combo (no duplication between sets)
   - Traceable: prefix `auto_combo{XX}_{dimension_short}`
   - No real PII
   - Computed values must match the formula

---

### Step 3P: Generate Test Data — PIPELINE Mode (running live in the browser)

> Perform when mode = PIPELINE

1. **Open the browser with MCP:**
   ```
   browser_navigate → application URL
   browser_resize → 1920 × 1080
   ```

2. **Loop through each combination set:**

   ```
   FOR each combo in matrix:
     FOR each module in chain:
       1. Navigate → module URL
       2. browser_snapshot → confirm the state
       3. Fill in data per the combo set:
          - Dimension fields → select the value per the combo
          - Supporting fields → generate random + traceable
       4. Submit / Save
       5. browser_wait_for → confirm success
       6. browser_snapshot → capture the result
       7. Extract output (ID, code...) → save it for the next module
     END FOR
     
     // At the final module — verify the output
     8. Capture the final record / output
     9. browser_take_screenshot → evidence
     10. Record: combo_id, created_ids, template_found, formula_verified
   END FOR
   ```

3. **Handle errors in the pipeline:**

   | Error | How to handle |
   |-----|-----------|
   | Submit fail (validation) | Screenshot → log → skip the combo → notify the user |
   | Module loads slowly | `browser_wait_for` with an increasing timeout |
   | Session expired | Re-login → retry from the failed module |
   | Duplicate data | Regenerate unique data, retry |
   | Invalid combo (constraint) | Skip → mark "INVALID" in the report |

4. **Pipeline limits:**
   - Maximum **30 combination sets** per run (to avoid timeouts)
   - If > 30 → split into batches, ask the user "Continue with the next batch?"
   - After every 10 sets → report progress to the user

---

### Step 4: Package the Output & Report

#### Output for GENERATE Mode:

Create the artifact file(s) in the format the user requested:

**JSON (default):**
```json
{
  "feature": "Partner payment record",
  "generated_at": "2026-04-15T17:00:00Z",
  "strategy": "pairwise",
  "total_combinations": 20,
  "dimensions": ["partner_type", "payment_type", "tax_type", "debt_type", "asset_source"],
  "data_sets": [
    { "combination_id": "COMBO_01", "dimensions": {...}, "module_data": {...}, "expected_output": {...} },
    { "combination_id": "COMBO_02", ... }
  ]
}
```

**Markdown Table:**
```markdown
| Combo | Partner | Payment | Tax | Debt | Source | Partner Name | Amount | Expected Template | Expected Total |
|-------|---------|-----------|------|---------|-------|-------------|--------|------------------|----------------|
| 01 | Organization | VND | VAT | Standard | Fund A | auto_c01_tc | 100M | BB_TC_VND_VAT | 110M |
| 02 | ... | ... | ... | ... | ... | ... | ... | ... | ... |
```

**Code (TypeScript example):**
```typescript
// test-data/payment-record.data.ts
export const combinatorialData = [
  {
    id: 'COMBO_01',
    partner: { name: `auto_combo01_${Date.now()}`, type: 'Organization', taxId: '0123456789' },
    payment: { currency: 'VND', amount: 100_000_000 },
    tax: { type: 'VAT', rate: 10 },
    expected: { template: 'BB_TC_VND_VAT', total: 110_000_000 },
  },
  // ... more combos
];
```

#### Output for PIPELINE Mode:

```markdown
## Pipeline Execution Report

| # | Combo | Status | Module 1 ID | Module 2 ID | Module 3 ID | Output Template | Formula ✓ | Screenshot |
|---|-------|--------|------------|------------|------------|-----------------|-----------|------------|
| 1 | COMBO_01 | ✅ PASS | PTR-001 | PAY-001 | TAX-001 | BB_TC_VND_VAT | ✅ Match | combo01.png |
| 2 | COMBO_02 | ✅ PASS | PTR-002 | PAY-002 | TAX-002 | BB_TC_USD_PIT | ✅ Match | combo02.png |
| 3 | COMBO_03 | ❌ FAIL | PTR-003 | PAY-003 | — | — | — | combo03_fail.png |

### Summary
- ✅ Passed: 18/20
- ❌ Failed: 2/20 (COMBO_03: Tax module validation error, COMBO_17: Timeout)
- 📊 Data created: 18 partners, 18 payments, 18 tax configs, 18 records
```

---

## Data Rules (MANDATORY)

| # | Rule | Description |
|---|------|-------|
| 1 | **Unique per combo** | Each combination set uses its own data — not shared between combos |
| 2 | **Traceable** | Prefix: `auto_combo{XX}_{dimension_short}_{timestamp}` |
| 3 | **Dimension values exact** | Dimension values MUST match exactly what is in the matrix — NOT random |
| 4 | **Supporting fields random** | Fields not belonging to a dimension → generate random + unique |
| 5 | **Computed values verified** | Computed values must match the formula in the matrix |
| 6 | **No real PII** | Do NOT use real personal data |
| 7 | **Include expected output** | Each combo MUST have an expected template + formula + computed values |

---

## PROHIBITED

| ❌ Do not do | ✅ Correct replacement |
|-------------------|-----------------|
| Random dimension values | Take dimension values exactly from the matrix |
| Hardcode identical data for every combo | Unique data per combo with a traceable prefix |
| Skip the expected output | Each combo MUST have an expected template + values |
| Run the pipeline > 30 combos per run without asking | Split into batches of 30, ask the user to continue |
| Skip failed combos without reporting | Log everything: which combo failed, why, screenshot |
| Read `.env` to obtain credentials | Ask the User or use an existing fixture |

---

## Final checklist

- [ ] Read and fully parsed the combination matrix
- [ ] Classified fields: dimension / supporting / computed / reference
- [ ] Generated data is unique per combo + traceable
- [ ] Dimension values are 100% correct against the matrix
- [ ] Computed values match the formula
- [ ] (PIPELINE) Screenshot evidence for each combo
- [ ] (PIPELINE) Pass/fail report for each combo
- [ ] Contains no real PII
- [ ] Output file matches the format the user requested
