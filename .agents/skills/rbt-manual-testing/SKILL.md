---
name: rbt-manual-testing
description: Skill for generating manual test cases with 2 modes — QUICK (fast generation from requirements) and FULL RBT (6-step AI-RBT process with risk assessment). Master skill for every manual test case task.
---

# RBT Manual Testing

## Description

This is the **Master Skill** for every manual test case generation task. The skill provides **2 operating modes** to fit any scale of requirement:

| Mode | When to use | Duration |
|------|-------------|----------|
| **QUICK** | Simple module, need TCs fast, clear requirements | Single pass (no user wait) |
| **FULL RBT** | Complex module, risk analysis needed, large system | 6 sequential steps (with checkpoints) |

**Core principles:**
- **Human Strategy:** Humans define the strategy, risk level, and standards
- **AI Execution:** AI performs the analysis, writes TCs, and reviews for gaps
- **Human Verification:** Humans review the results before finalizing

---

## When to Use

Use this skill when:

- Generating manual test cases from requirements / user stories
- Analyzing requirements to detect ambiguity
- Decomposing a system into modules / features
- Building a traceability matrix
- Applying Risk-Based Testing (risk assessment for test cases)
- Standardizing test cases into a Markdown table (Jira/Excel format)
- Quickly generating test cases from simple requirements

Do **NOT** use this skill when:

- You need to generate automation code → use `qa-automation-engineer`
- You need to inspect the DOM / generate locators → use `ui-debug-agent` / `smart-locator-agent`
- You only need to generate test data → use `test-data-generator`

---

## Mode Routing — How to choose a mode

The agent selects the mode automatically based on **trigger keywords** and **context**:

### → QUICK Mode

Activate when:
- The user uses the `generate-testcases-from-requirements` workflow
- The user says: "generate test cases quickly", "create TCs from this requirement", "write test cases for this form..."
- Requirements are already clear, scope is small (1 module / 1 feature)
- The user does not request risk analysis or a formal process

### → FULL RBT Mode

Activate when:
- The user uses the `generate-manual-testcases-rbt` workflow
- The user says: "6-step process", "RBT analysis", "generate full test cases", "generate a rigorous TC set"
- Scope is large (many modules, complex system)
- The user requests a Traceability Matrix or Risk Level assessment
- Requirements are unclear and require Ambiguity analysis

### → When unclear

If the mode cannot be determined, the agent **asks the user**:
```
Which mode would you like to generate test cases in?
1. QUICK — Fast generation from requirements (no analysis step)
2. FULL RBT — Full 6-step process (analysis → decomposition → RBT → TC generation)
```

---

# Mode 1: QUICK — Fast Test Case Generation

## Purpose

Generate **fast, sufficiently high-quality** test cases from clear requirements/user stories, suitable for simple modules or when immediate results are needed.

## Process (single pass)

**The agent must:**

1. **Read and understand the provided requirements**
2. **Identify the main flows:**
   - Happy Path (main flow)
   - Negative Path (wrong or missing data)
   - Boundary Cases (boundary values)
3. **Apply test case design techniques** automatically:
   - **Equivalence Partitioning (EP):** Split input into equivalent groups
   - **Boundary Value Analysis (BVA):** Test values at the boundaries
   - **Decision Table:** List condition combinations (if there are many rules)
   - **State Transition:** Test state transitions (if there is a workflow)
4. **Field-Level Validation:**
   - List **all input fields** on the form/UI
   - Generate validation test cases **for EACH field individually** based on its own characteristics
   - Apply the validation checklist per field type (see the Field-Level Validation table below)
   - Do **NOT** merge validation of multiple fields into a single test case
5. **Generate test cases** with all fields:
   - TC ID (format: `[PROJECT]_[MODULE]_TC_[NUMBER]`)
   - Module
   - Test Case Title / Test Scenario
   - Pre-conditions
   - Test Steps (numbered)
   - Expected Results (numbered correspondingly)
   - Test Data (**must be concrete**, no placeholders)
   - Priority (Critical / High / Medium / Low)
6. **Export a standard Markdown table**, ready to copy into Excel/Jira

## Output Table

```
| TC ID | Module | Test Scenario | Pre-Condition | Test Steps | Test Data | Expected Result | Priority |
```

## Test Data Rules (apply to both modes)

```
❌ Wrong: "Enter a valid code"
✅ Right: "Enter code: KH-2026-0012"

❌ Wrong: "Enter a valid email"
✅ Right: "Enter email: test_customer_01@domain.com"

❌ Wrong: "Enter a value exceeding the limit"
✅ Right: "Enter 256 characters into the Name field (max: 255)"
```

## Field-Level Validation Table (apply to both modes)

When the form/UI has input fields, the agent **MUST** list each field and generate separate validation TCs by type:

| Field Type | Validation to test |
|---|---|
| **Text (Name, Address...)** | Required/Optional · Min length · Max length · Whitespace-only · Special characters (`<>&"'`) · XSS injection (`<script>alert(1)</script>`) · SQL injection (`' OR 1=1--`) · Unicode/Emoji · Leading/trailing spaces |
| **Email** | Valid format (`user@domain.com`) · Missing `@` · Missing domain · Invalid domain · Multiple `@` · Special characters before `@` · Max length · Case sensitivity · Email already exists (if unique) |
| **Phone** | Digits only · Valid prefix (e.g., `+84`, `0`) · Min/Max length · Mixed-in letters · `-`, `.`, spaces · Invalid area code |
| **Date / DateTime** | Correct format (dd/MM/yyyy, ISO...) · Non-existent date (`31/02`, `30/02`) · Leap year (`29/02/2024`) · Past / future date (per business rule) · Min/max date value · Timezone (if applicable) |
| **Number / Currency** | Min/Max value · Negative number · Zero · Decimals · Non-numeric characters · Overflow (extremely large number) · Leading zeros · Currency format (commas, dots) |
| **Dropdown / Select** | Default value · All valid options · Disabled option · Change selection · Required validation (nothing selected) |
| **Checkbox / Radio** | Default state · Check/Uncheck · Required validation · Radio group (only one selectable) |
| **File Upload** | Valid/invalid file type · Max size · Empty file (0 KB) · File name with special characters · Multiple files (if allowed) · Drag-and-drop vs. select button |
| **Password** | Min/Max length · Special character requirement · Uppercase/lowercase requirement · Digit requirement · Copy-paste blocked? · Show/hide password · Confirm password matches/does not match |
| **Textarea** | Max length · Line breaks · HTML tags · Resize (if the UI allows) · Character counter (if present) |

> **Principle:** Each field has its own characteristics → its own validation. The agent MUST analyze each field before generating TCs, and must not reuse a single validation set for all fields.

## Anti-Patterns (QUICK Mode)

- ❌ Generating generic / placeholder test data
- ❌ Only Happy Path, missing Negative/Boundary
- ❌ Ignoring validation rules in the requirements
- ❌ Vague Test Steps ("enter data" → must state what to enter and where)
- ❌ Merging validation of multiple fields into one test case → each field must have its own validation TC
- ❌ Reusing a single validation set for all fields (each field type has its own checklist)
- ❌ Skipping security validation (XSS, SQL injection) for text fields

---

# Mode 2: FULL RBT — 6-Step AI-RBT Process

## Purpose

A rigorous, sequential process for complex modules. It includes Ambiguity analysis, system decomposition, a Traceability Matrix, Risk Level assessment, and detailed test case generation.

> ⚠️ **IMPORTANT:** This process **MUST run sequentially** step by step. Do NOT combine multiple steps into one run. Each step must be completed and confirmed by the user before moving to the next.

> [!NOTE]
> **2 separate usage flows:**
> - **Skill flow:** Follow the general guidance below. There is no need to read the `prompt.txt` file.
> - **Copy-Paste flow:** The QA team copies sequentially from `plans/manual/01_context_and_roleplay/prompt.txt` to `plans/manual/06_template_mapping/prompt.txt` into the AI chat, one step at a time.

### Step 1: Context & Role-play (Initialize context)

**Purpose:** Establish the Senior QA Engineer role and load the project context.

**The agent must:**
1. Ask the user to provide:
   - Project / feature name
   - Description of the current system
   - MVP testing objective
   - Requirement documents (Requirements, User Stories, Figma link, PDF...)
2. Read the documents carefully and confirm understanding of the context
3. Summarize the testing scope
4. **Wait for user confirmation** before moving to Step 2

**Output:** Confirmation of context understanding + testing scope summary.

---

### Step 2: Analysis & QnA (Requirement analysis)

**Purpose:** Analyze the documents to detect ambiguities, gaps, and contradictions.

**The agent must:**
1. Identify the flows:
   - Happy Path (main flow)
   - Alternate Paths (branching flows)
   - Exception Paths (exceptional flows)
2. Detect Ambiguities:
   - Missing requirements (no textbox length, timeout, or disconnect behavior specified...)
   - Contradictory requirements
   - Unclear requirements
3. Ask numbered Q&A questions (Q1, Q2...) for the user/PO/BA to answer, each with context and an assumption if unanswered
4. **STOP — Wait for the user to answer** the questions before continuing

**Output:** List of flows + Ambiguities + Q&A questions.

> [!IMPORTANT]
> **This is the most critical bottleneck.** If the agent skips this step and guesses the logic, the test cases will be seriously wrong. The agent MUST stop and wait for the user's response.

---

### Step 3: Decomposition (System decomposition)

**Purpose:** Split a complex feature into small, manageable Modules / Sub-modules.

**The agent must:**
1. Decompose in one of two ways:
   - **By UI:** Header, Data Table, Form popup, Sidebar...
   - **By flow:** Create flow, Edit flow, Delete flow...
2. Briefly describe the function of each Module
3. Indicate the Dependencies between Modules

**Output:** List of Modules/Sub-modules + Dependencies.

---

### Step 4: Traceability (Ensure coverage)

**Purpose:** Establish a traceability matrix to ensure 100% of the requirements are covered by test scenarios.

**The agent must:**
1. Map each Module/Rule to a Requirement ID (REQ-01, REQ-02...)
2. Cross-check whether any requirement is missing from the decomposition list (Gap Analysis)
3. List High-Level Test Scenarios for each Module, focusing on:
   - Security / authorization
   - UI Validation
   - Business Logic
   - Data Integrity
   - Error Handling
4. **Wait for the user to review** the scenario list before generating detailed test cases

**Output:** Traceability Matrix + High-Level Test Scenarios.

> [!WARNING]
> **Human Checkpoint:** The user needs to review the scenario list to add specific cases the AI might have missed. This is the human-performed risk assessment step.

---

### Step 5: RBT & TC Generation (Detailed test case generation)

**Purpose:** Generate detailed test cases following the Risk-Based Testing strategy.

**The agent must:**
1. Assess the Risk Level for each Module:
   - **High Risk:** Test thoroughly, many cases (important business logic, money-related, security)
   - **Medium Risk:** Test moderately
   - **Low Risk:** Basic testing, happy path
2. Generate test cases with all fields:
   - Module / Sub-module
   - Test Case Title
   - Pre-conditions
   - Test Steps (numbered)
   - Expected Results (numbered correspondingly)
   - Test Data (**must be concrete**, no generic placeholders)
   - Priority
3. Cover a diverse range:
   - Happy Path
   - Negative Path (boundary values, exceeding character limits)
   - Edge Cases (timeout, disconnection...)
4. **Field-Level Validation:**
   - List **all input fields** on the form/UI under test
   - Generate validation TCs **for EACH field individually** based on its own characteristics
   - Reference the **Field-Level Validation Table** in the QUICK Mode section to choose the appropriate validation
   - Do **NOT** merge validation of multiple fields into a single TC
5. Apply the appropriate **test case design techniques**:
   - **Equivalence Partitioning:** Split input into equivalent groups, test a representative of each group
   - **Boundary Value Analysis (BVA):** Test values at the boundaries (min, min+1, max-1, max)
   - **Decision Table:** List condition combinations → outcome (for multi-condition logic)
   - **State Transition:** Test valid + invalid state transitions (for workflows)
6. If there are too many scenarios → generate one Module at a time, asking the user to continue

**Output:** Detailed Test Case list with Risk Level.

---

### Step 6: Template Mapping (Format standardization)

**Purpose:** Package the test cases into a standard Markdown table, ready to copy into Excel/Jira.

**The agent must:**
1. Standardize all test cases into a Markdown table:

```
| TC ID | Module | Risk Level | Test Title | Pre-Condition | Test Steps | Expected Result | Priority | Test Data |
```

2. Table rules:
   - TC ID follows a consistent format (e.g., `CRM_CUST_TC_001`)
   - Number the Test Steps and Expected Result, using `<br>` for line breaks within a cell
   - **NEVER omit** any test case generated in Step 5
   - If too long → split into Part 1, Part 2... and ask the user to continue
3. Export the output as an artifact (`test_cases_<module>.md`)

**Output:** A complete Markdown Test Case table.

---

## Anti-Patterns (FORBIDDEN — apply to both modes)

- ❌ Combining multiple steps into one run in FULL RBT (it MUST be sequential)
- ❌ Guessing the business logic before asking the user (Step 2 - FULL RBT)
- ❌ Skipping the Ambiguity analysis step (FULL RBT)
- ❌ Generating generic / placeholder test data
- ❌ Abbreviating or omitting test cases when mapping into the table
- ❌ Generating all test cases at once for a large system (must split by module)
- ❌ Only Happy Path, missing Negative/Boundary cases (QUICK)
- ❌ Vague Test Steps that do not state the input data clearly
- ❌ Merging validation of multiple fields into one test case → each field must have its own validation TC
- ❌ Reusing a single validation set for all fields (Email ≠ Phone ≠ Date ≠ Text)
- ❌ Skipping security validation (XSS, SQL injection) for text/textarea fields
- ❌ Not listing the fields before generating validation TCs

---

## Prompt Templates

Sample prompt templates for the FULL RBT process are located at:

```
plans/manual/
├── 01_context_and_roleplay/prompt.txt
├── 02_analysis_and_qna/prompt.txt
├── 03_decomposition/prompt.txt
├── 04_traceability/prompt.txt
├── 05_rbt_and_tc_generation/prompt.txt
└── 06_template_mapping/prompt.txt
```

The agent should read the corresponding prompt template **before** performing each step (FULL RBT mode).

QUICK mode does not require reading prompt templates — the agent applies the EP/BVA/Decision Table techniques directly.

---

## Output Format

### QUICK Mode

| Output | Description |
|--------|-------------|
| Markdown TC table | Complete Test Cases, ready to copy into Excel/Jira |

### FULL RBT Mode

| Step | Output |
|------|--------|
| 1 | Context confirmation |
| 2 | Flows + Ambiguities + Q&A questions |
| 3 | Module Decomposition + Dependencies |
| 4 | Traceability Matrix + High-Level Scenarios |
| 5 | Detailed Test Cases (Risk Level + Test Data) |
| 6 | Standard Markdown table (Jira/Excel ready) |

All output must be in **English**, in **Markdown** format, using an **artifact** if the content is long.
