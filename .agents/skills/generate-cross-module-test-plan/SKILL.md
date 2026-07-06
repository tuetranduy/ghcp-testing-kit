---
name: generate-cross-module-test-plan
description: Analyze a feature that flows through multiple sequential modules, build a Module Map + Dimension Catalog, and generate a combination matrix (Pairwise/Business-critical/Full Cartesian). Supports 2 modes — DOCUMENT (from documentation) and BROWSER (inspect the real DOM).
---

# generate-cross-module-test-plan — Cross-Module Analysis & Combination Matrix Generation

> **Use when:** The feature under test flows through **multiple sequential modules**, each module has several choices (dimensions), and the combination of choices determines the final output.

> **MANDATORY:** Before starting, you MUST load and carefully read:
> - **Skill:** `.agents/skills/qa-automation-engineer/SKILL.md` — Workflow routing + automation rules
> - **Skill:** `.agents/skills/requirements-analyzer/SKILL.md` — Requirements analysis

---

## When to use this workflow?

| Situation | Use? |
|------------|-------|
| Feature flows through **1 module/form** | ❌ Use `generate-manual-testcases-rbt` |
| Feature flows through **multiple modules**, each module **independent** | ⚠️ Use `generate-application-test-plan` |
| Feature flows through **multiple SEQUENTIAL modules**, output depends on the **combination of conditions** | ✅ **This workflow** |
| Need a **combination matrix** (Pairwise / multi-dimensional Decision Table) | ✅ **This workflow** |

---

## 2 Modes

| Mode | When to use | Main input |
|------|-------------|-------------|
| **DOCUMENT** (default) | User provides a document/spec describing the modules + business rules | `.md`, `.doc` file, Jira ticket, or text description |
| **BROWSER** | User provides a URL and wants the agent to inspect the real DOM | Application URL + credentials (if needed) |

> The agent selects the mode automatically:
> - User provides a file/text description → **DOCUMENT**
> - User provides a URL or says "inspect", "open the app", "look in the browser" → **BROWSER**
> - User provides both → Prefer **BROWSER**, use the document to cross-reference
> - If unclear → ask the user

---

## Inputs required from the User

| Input | Required | Description |
|-------|----------|-------|
| **Feature / flow name** | ✅ | E.g.: "Partner payment record" |
| **Requirements document** (DOCUMENT mode) | ✅ | `.md` file, Jira ticket, user story, or a text description |
| **Application URL** (BROWSER mode) | ✅ | So the agent can inspect the real DOM |
| **List of participating modules** | ⚠️ Recommended | If missing → the agent extracts them from the document/browser |
| **List of dimensions** | ⚠️ Recommended | E.g.: partner type, tax type... If missing → the agent extracts them |
| **Business rules / formulas** | ❌ Optional | If provided → the agent maps them into the matrix |
| **Credentials** (BROWSER mode) | ❌ | If the app requires login |
| **Matrix strategy** | ❌ | `pairwise` (default), `business-critical`, or `full-cartesian` |

---

## Steps

### Step 1: Module Recon — Explore each Module

#### DOCUMENT Mode:

1. **Read the document** provided by the user (local file → read the file, URL → fetch the webpage, inline → parse directly)
2. **Extract the list of modules** from the document:
   - Find the sections describing each step/module in the flow
   - Determine the module order (which module comes first, which comes after)
   - Determine the fields/controls mentioned for each module
3. **If the document is unclear** → ask the user for specific additional information

#### BROWSER Mode:

1. **Receive the list of modules** from the user or discover it through navigation
2. **For each module** in the chain:
   ```
   browser_navigate → module URL
   browser_resize → 1920 × 1080
   browser_wait_for → page load
   browser_snapshot → collect the DOM
   ```
3. **Collect for each module:**
   - Module name (from the page title / breadcrumb)
   - Fields / Controls (input, select, radio...)
   - Selectable values (open the dropdown → read the options)
   - Validation rules (required, format, min/max)

#### Step 1 Output — Module Inventory (both modes):

```markdown
| # | Module | URL / Path | Main Inputs | Key Dimensions | Output | → Next Module |
|---|--------|-----------|-------------|---------------|--------|---------------|
| 1 | Partner Management | /partners | Name, Tax ID, Type | **Partner type** (3 values) | Partner ID | → Payment |
| 2 | Create Payment | /payments/new | Amount, Type | **Payment type** (2 values) | Payment ID | → Tax |
| ...| ... | ... | ... | ... | ... | ... |
```

---

### Step 2: Data Flow & Dimension Extraction

> Combine Data Flow Mapping + Dimension Extraction into 1 step to reduce context switching.

1. **Determine the Data Flow** between modules:
   - What does module A **output**?
   - How does that output become the **input/condition** for module B?

2. **Record the Dependencies Matrix:**

   ```markdown
   | Target module | Depends on | Dependent field | Dependency type |
   |-------------|--------------|-----------------|----------------|
   | Payment | Partner | Partner.type | Filters payment options |
   | Tax | Partner + Payment | Partner.type, Payment.currency | Determines the tax type |
   ```

3. **Extract Dimensions** — the variables that determine the output:

   ```markdown
   | # | Dimension | Source module | Possible values | # values |
   |---|------------------|-------------|----------------|-----------|
   | D1 | Partner type | Partner | Organization, Individual, Household Business | 3 |
   | D2 | Payment type | Payment | VND, USD | 2 |
   | D3 | Tax type | Tax | PIT, VAT, Contractor, Tax-exempt | 4 |
   ```

4. **Compute the total potential combinations:**
   ```
   Full Cartesian: D1 × D2 × D3 × ... = total combination sets
   ```

5. **Identify constraints** (invalid combination sets):
   ```markdown
   | Constraint | Description | Sets excluded |
   |-----------|-------|-----------|
   | C1 | Individual + USD → no Contractor | 1 set |
   ```

6. **Present the analysis results to the user:**
   - Module Inventory
   - Dependencies Matrix
   - Dimension Catalog
   - Constraints
   - **Continue automatically** to Step 3 (do NOT stop at a checkpoint — only stop if the user proactively says something is wrong)

---

### Step 3: Generate the Combination Matrix (CORE OUTPUT ⭐)

The agent supports **3 strategies** — the user chooses or the agent proposes:

#### 3A. Pairwise Testing (Default — RECOMMENDED)

> Ensures every **pair of 2 values** from any 2 dimensions is tested at least once.
> Significantly reduces the number of combination sets while still covering 100% of pairs.

**How to do it — you MUST use a script, do NOT compute manually:**

1. The agent **generates a Python script** using the `allpairspy` library:

   ```python
   # pairwise_generator.py
   from allpairspy import AllPairs
   
   dimensions = {
       "D1_partner_type": ["Organization", "Individual", "Household Business"],
       "D2_payment_type": ["VND", "USD"],
       "D3_tax_type": ["PIT", "VAT", "Contractor", "Tax-exempt"],
       # ... add dimensions
   }
   
   # Constraints (invalid sets)
   def is_valid(row):
       # Example: Individual + USD → no Contractor
       if len(row) >= 3:
           if row[0] == "Individual" and row[1] == "USD" and row[2] == "Contractor":
               return False
       return True
   
   values = list(dimensions.values())
   keys = list(dimensions.keys())
   
   print(f"| # | {' | '.join(keys)} |")
   print(f"|{'---|' * (len(keys) + 1)}")
   for i, combo in enumerate(AllPairs(values, filter_func=is_valid)):
       row = ' | '.join(str(v) for v in combo)
       print(f"| {i+1} | {row} |")
   ```

2. The agent **runs the script** → reads the output → formats it into a Markdown table
3. If `allpairspy` is not installed → `pip install allpairspy` before running

> **PROHIBITED** for the agent to compute pairwise manually — an LLM cannot guarantee mathematical correctness.

#### 3B. Business-Critical Only

> Select only the **most important** combination sets based on business risk.

**Selection criteria:**
- The most common sets in practice (per user confirmation)
- Sets involving money, tax, or financial decisions → High Risk
- Boundary combination sets (edge cases between types)

**Count:** Typically 8-15 sets. The agent proposes → the user confirms.

#### 3C. Full Cartesian

> Test ALL valid combination sets. Use when the total ≤ 50 or the user requests it.

#### Step 3 Output — Matrix Table:

```markdown
## Combination Matrix (Pairwise — N sets)

| # | D1: Partner | D2: Payment | D3: Tax | D4: Debt | Risk |
|---|------------|---------------|---------|------------|------|
| 1 | Organization | VND | VAT 10% | Standard | High |
| 2 | Organization | USD | PIT 10% | Advance | High |
| 3 | Individual | VND | PIT 10% | Standard | High |
| ... | ... | ... | ... | ... | ... |
```

> **Note on Expected Template / Formula:**
> - If the user provides business rules → add an "Expected Output" column to the table
> - If NOT → **do NOT add this column**, leave it for the next workflow to handle
> - Do NOT fill in `[User confirmation needed]` — if unknown, leave it blank, do not create noise

---

### Step 4: Package the Output & Generate Data (CHECKPOINT ⏸️)

> This is the **ONLY checkpoint** — the agent stops here to wait for user confirmation.

1. **Generate the main artifact:**

   **File: `cross_module_test_plan_<feature>.md`**
   - Module Inventory (Step 1)
   - Dependencies Matrix (Step 2)
   - Dimension Catalog + Constraints (Step 2)
   - **Combination Matrix** (Step 3) — THIS IS THE MAIN OUTPUT
   - Risk Assessment for each combination set

2. **(Optional) Generate test data at the same time** if the user requests `--with-data`:

   For each combination set in the matrix, generate 1 data set:
   ```json
   {
     "combination_id": "COMBO_01",
     "dimensions": { "D1": "Organization", "D2": "VND", "D3": "VAT 10%" },
     "supporting_data": {
       "partner_name": "auto_c01_tc_<timestamp>",
       "tax_id": "0123456789",
       "amount": 100000000
     }
   }
   ```
   
   **Data Rules:**
   - Dimension values MUST be 100% correct from the matrix — NOT random
   - Supporting fields → random + unique + traceable (prefix `auto_combo{XX}`)
   - No real PII

3. **⏸️ STOP — Present to the user:**
   - The complete combination matrix
   - Ask: "Is any combination set missing? What needs adding?"
   - **Wait for user confirmation**

---

## Next steps after this workflow

| Goal | Next workflow |
|----------|-------------------|
| Generate **detailed test cases** for each combination set | `generate-manual-testcases-rbt` — input = the matrix |
| Create **real test data in the system** via a browser pipeline | `generate-combinatorial-test-data` — PIPELINE mode |
| Generate **automation scripts** | `generate-automation-from-testcases` — input = test cases |

---

## Real examples

### Example 1: E-Commerce Order (DOCUMENT mode)

**User says:** "Analyze the ordering flow: Select product → Select shipping → Payment → Confirmation"

**The agent does:**
1. Extract 4 modules from the description
2. Dimensions: Product type (3), Shipping (3), Payment (4) = 36 full sets
3. Run the pairwise script → reduce to ~12 sets
4. Output: A 12-set matrix + Module Map

### Example 2: Insurance Contract (BROWSER mode)

**User says:** "Inspect the insurance app at https://example.com, the contract creation flow"

**The agent does:**
1. Navigate through 3 modules, snapshot each module
2. Extract dimensions from the DOM: Insurance type (3), Plan (4), Term (3), Payment method (2) = 72 full sets
3. Run the pairwise script → reduce to ~15 sets
4. Output: A 15-set matrix + Data Flow

---

## PROHIBITED

| ❌ Do not do | ✅ Correct replacement |
|-------------------|-----------------| 
| Compute pairwise manually (greedy/IPOG) | Generate a Python script + `allpairspy` → run the script |
| Guess dimension values with no source | DOCUMENT: extract from the document, BROWSER: inspect the DOM |
| Fill in Expected Template/Formula when unknown | Leave blank or ask the user — do NOT write `[Confirmation needed]` |
| Default to Full Cartesian when dimensions are large | Use Pairwise — reduce 80-90% of combination sets |
| Ignore constraints (invalid combination sets) | You must identify and remove invalid combinations |
| Load too many skills (>2) into context | Only load the 2 mandatory skills, add others when needed |

---

## Final checklist

- [ ] Collected information for EACH module (via document or browser)
- [ ] Built the Dependencies Matrix between modules
- [ ] Fully extracted dimensions + values + constraints
- [ ] Chose the appropriate matrix strategy
- [ ] **(Pairwise)** Generated and ran the script — NOT computed manually
- [ ] The matrix contains only valid combination sets (constraints removed)
- [ ] The user confirmed the matrix at Step 4
- [ ] The output artifact is saved in the correct project location
