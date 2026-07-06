---
name: generate-requirements-from-website
description: Generate Requirements content from a provided website module
---

# Workflow: Generate Requirements from Website Module

> **MANDATORY SKILL:** You MUST load and carefully read the content of the **`requirements-analyzer`** skill (at `.agents/skills/requirements-analyzer/SKILL.md`) to learn the standard format of the Requirements document before starting this task.

This workflow helps you analyze a provided module or web page and generate a detailed, accurate Requirements document to support testing or development.

## Steps:

1. **Information Gathering:**
   - Carefully read the guidance from the **`requirements-analyzer`** skill to understand the expected output.
   - Collect the website URL, module name, or the description/image provided by the user.
   - If necessary, ask the user about login credentials or any special states to be aware of.

2. **Recon & Investigation:**
   - Use the browser tools (Playwright MCP) or fetch the webpage to access the requested web module.
   - Carefully inspect the HTML structure, DOM, input forms, interactive elements (buttons, links), and validation messages.
   - *Note: Do not guess fields that are not visible on the actual interface.*

3. **Analyze UI & Interactions:**
   - Analyze the user flows.
   - Record static and dynamic data fields (e.g., TextBox, Dropdown, Checkbox).
   - Record the business rules shown on the interface, such as: mandatory fields, valid formats (email, phone number), or character limits.

4. **Draft Requirements:**
   - Based on the collected data, create a descriptive document that includes:
     * **Overview:** The purpose of the module/page.
     * **Functional Requirements:** A list of actions the user can perform (e.g., Login, Add, Delete...).
     * **Field Specifications:** A detailed table of each UI component (Field name, Type, Required/Optional, Data constraints).
     * **Business/User Flows:** The steps to complete a main function.
     * **Non-functional Requirements (if observable):** Compatibility, static performance.

5. **Review & Delivery:**
   - Format the document with clear Markdown.
   - Present all content in clear, professional, and easy-to-understand **English**.
   - Use the Artifact feature if the document is long, so the user can conveniently store or export it.