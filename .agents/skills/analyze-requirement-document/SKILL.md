---
name: analyze-requirement-document
description: Analyze a requirement document (Jira ticket, .doc, user story) — produce a detailed analysis document, does NOT generate test cases.
---

> **MANDATORY SKILL:** You MUST load and carefully read the content of the **`requirements-analyzer`** skill (at `.agents/skills/requirements-analyzer/SKILL.md`) to understand the standard way to analyze requirements before starting.

# Workflow: Analyze Requirement Document

This workflow analyzes requirement documents (Jira tickets, .doc files, user stories, design mockups) and produces a detailed analysis document. **It does NOT generate test cases** — it focuses only on understanding, decomposing, and detecting risks/ambiguities in the requirement.

## When to use

- The user provides a Jira ticket (.doc) or requirement document and asks to "analyze" it
- The user wants to clearly understand the scope, acceptance criteria, and dependencies before writing tests
- The user needs a list of ambiguities to clarify with the PO/BA
- The user says: "analyze the requirement", "review the requirement", "analyze this ticket"

## Input

The agent needs to collect from the user:

| # | Input | Required | Description |
|---|---|---|---|
| 1 | **Requirement document** | ✅ | A .doc, .md file, Jira URL, or text describing the requirement |
| 2 | **Mockup/Screenshot** | ⭕ Recommended | An image of the UI design, wireframe, or current screenshot |
| 3 | **Related tickets** | ⭕ Optional | Dependent or related tickets (dependencies) |
| 4 | **Additional context** | ⭕ Optional | Information about the current system, business domain |

> [!NOTE]
> If the user provides only a .doc file without a mockup, the agent must still analyze it fully based on the document content. If a mockup/screenshot is available, the agent analyzes the UI in more detail.

## Steps

### Step 1: Gather and comprehend (Information Gathering)

1. **Read the requirement document** provided by the user (.doc, .md file, or URL)
   - If the .doc file is in HTML format (exported from Jira): parse the HTML to extract the content
   - Identify: Ticket ID, Type, Priority, Status, Reporter, Assignee, Fix Version, Sprint, Labels
2. **Read the mockup/screenshot** if available — analyze the UI layout, components, fields
3. **Check related tickets** if present in the same folder or provided by the user
   - Read and summarize the dependencies
4. **Confirm** you have grasped the context → continue with the analysis

### Step 2: Extract the core information (Core Analysis)

1. **Ticket Overview** — Metadata table (ID, Type, Priority, Status, Sprint, Assignee...)
2. **User Story** — Extract the "As a... I want... So that..." format
3. **Scope** — Clearly identify the affected modules/pages/components
4. **Acceptance Criteria** — Decompose each AC into logical groups, including:
   - A detailed description of each AC
   - A comparison table (if there are new columns, new fields, new rules)
   - A clear distinction between **default vs optional** (if applicable)

### Step 3: Analyze the UI from the Mockup (if available)

If the user provides a mockup/screenshot:

1. **Describe the layout** — Breadcrumb, header, sidebar, main content, footer
2. **List the components** — Tables, forms, modals, buttons, dropdowns, tabs
3. **Detail the fields** — Field name, type (input/dropdown/date picker), label, placeholder
4. **Compare** the mockup with the document — detect inconsistencies
5. **Capture observations** into the carousel in the artifact (if the images are available)

### Step 4: Analyze Dependencies

1. Identify related tickets/features (referenced in the AC or comments)
2. Read and summarize the content of dependent tickets
3. If a dependency has its own mockup → analyze the UI in detail (fields, modals, interactions)
4. Consolidate the **Business Rules** from all requirements + mockups
5. Clearly mark which rule comes from the main ticket vs a dependent ticket

### Step 5: Detect Ambiguities & Risks (Key focus)

> [!IMPORTANT]
> This is the **highest-value** part of the workflow — detecting what the requirement does NOT state clearly.

**5.1. Ambiguities:**

For each ambiguity, note clearly:
- **Code:** AMB-XX (numbered sequentially)
- **Question:** A clear description of what is unclear
- **Risk:** The impact if it is not resolved
- **Level:** 🔴 High / 🟡 Medium / 🟢 Low

Directions for detecting ambiguities:
- Vague keywords: "where applicable", "as needed", "similar to", "etc."
- Missing validation rules: min/max, format, required/optional
- Edge-case behavior: network error, concurrent access, empty data
- Inconsistency between the document and the mockup (column names, format, layout)
- Undefined thresholds/config (e.g. how many days counts as "approaching deadline"?)
- Conflict between old and new requirements

**5.2. Testing Risks:**

For each risk, note clearly:
- **Code:** RISK-XX
- **Risk name**
- **Description**
- **Mitigation** (how to reduce it)

### Step 6: Synthesis & Delivery

1. **State matrix** (if there are state transitions) — a table mapping state → behavior
2. **AC checklist** — Summarize all ACs as checkboxes, grouped by function
3. **Testing recommendations** — Suggest the top 10 things to focus on most when testing
4. **Export the artifact** — Save the entire analysis to a `.md` file

## Output Structure (Artifact Template)

The agent MUST export the artifact using the following structure:

```markdown
# 📋 Requirement Analysis: [TICKET-ID]
## [Ticket Title]

## 1. Ticket Overview
(Metadata table)

## 2. User Story
(As a... I want... So that...)

## 3. Scope
(Table listing the affected modules/pages)

## 4. Acceptance Criteria — Detailed Analysis
### 4.1. [AC Group 1]
### 4.2. [AC Group 2]
### 4.N. [AC Group N]

## 5. Dependencies
### 5.1. [Dependent ticket]
#### 5.1.1. [UI detail if a mockup exists]
#### 5.1.N. Consolidated Business Rules

## 6. Mockup/Screenshot Analysis
### 6.1. [Mockup 1]
### 6.N. [Mockup N]

## 7. Ambiguities & Risks
### 7.1. Ambiguities
(Table: #, Question, Risk, Level)
### 7.2. Testing Risks
(Table: #, Risk, Description, Mitigation)

## 8. State Matrix (if applicable)
(Table state → behavior)

## 9. Acceptance Criteria Summary (Checklist)
(Checkboxes grouped by function)

## 10. Testing Recommendations
(List of suggestions, NOT test cases)
```

## Important rules

- ❌ **Do NOT generate test cases** — this workflow only analyzes, it does not create TCs
- ❌ **Do NOT guess** business logic if the document does not state it clearly → put it in Ambiguities
- ❌ **Do NOT ignore comments** in the Jira ticket — comments often contain important additional information
- ✅ **You MUST read related tickets** if they are referenced in the AC
- ✅ **You MUST analyze the mockup** in detail if it is provided (fields, layout, interactions)
- ✅ **You MUST note inconsistencies** between the document and the mockup
- ✅ **You MUST write in English**, in Markdown format, and export the Artifact
- ✅ **You MUST copy images** into the artifacts folder if they need to be embedded in the artifact

## Relationship with other workflows

| After finishing the analysis | Next workflow |
|---|---|
| Need to generate test cases quickly | `generate-testcases-from-requirements` |
| Need to generate test cases thoroughly (6-step RBT) | `generate-manual-testcases-rbt` |
| Need to generate automation scripts | `generate-automation-from-testcases` |
| Need cross-module analysis | `generate-cross-module-test-plan` |
