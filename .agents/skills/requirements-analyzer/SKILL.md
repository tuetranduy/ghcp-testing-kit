---
name: requirements-analyzer
description: A skill for analyzing a web page/module and generating a standard Requirements Document / User Stories.
---

# Requirements Analyzer

This skill guides the conversion of a web page's UI or DOM/HTML structure into a clear, detailed requirements document for QA, Testers, and Developers.

## 1. Core objectives
- Build a requirements document that closely reflects the running system.
- Ensure consistency and coverage for both the Happy Path and Edge Cases (exceptional/error scenarios).
- Produce professionally formatted output (using an artifact structure).

## 2. Information extraction process
When asked to create Requirements from a web page:
1. **Layout Analysis:** Identify the Header, Footer, Sidebar, and Main Content sections.
2. **Collect Forms & Inputs:**
   - Find all input fields (`input`, `select`, `textarea`).
   - Record the `type` attribute (text, email, password, number), `required`, `maxlength`, `minlength`, `pattern`.
3. **Collect Interactive Elements (Buttons/Links/Actions):**
   - Identify the function of each button (Save, Submit, Cancel, Delete, Edit).
   - Note the alerts and messages (Alerts, Toasts, Validation Messages) that appear on error interactions.
4. **Extract Workflows:**
   - The dependencies between components (e.g., the Submit button is only enabled once the "I agree" checkbox is checked).

## 3. Output Requirements Document Structure (Output Format)
The document should be formatted in professional Markdown or saved as an artifact (`requirements_spec.md`).

**Mandatory content:**

### 3.1. Overview
A summary of the feature and the purpose of the web page/module.

### 3.2. Functional Requirements
Split into **User Stories** or **Use Cases**:
- **Feature name** (e.g., Login feature)
- **Description:** "As a user, I want... so that I can..."
- **Acceptance Criteria:** Clearly state the conditions that must be satisfied.

### 3.3. Field Specifications
This is the core section for the Automation Tester:
* Use a Markdown table to list:
  - Field Name (Label)
  - Type (UI Type)
  - Validation Rules (Required / Default / Length limit).
  - Notes.

### 3.4. Business Rules & Validations
List in detail the expected Validation Messages when the user enters invalid data.

## 4. Strict Rules
- Always write in **English**.
- Do not infer complex business requirements without evidence from the UI. If logic is missing, list it under the "Questions/Clarifications for PO-User" section.
- If Playwright MCP is available, prefer opening a real browser to screenshot/capture the UI when needed.
