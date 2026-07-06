# Step 3: Decomposition Strategy


---

## Purpose

Guide the AI to break a complex system / feature into multiple manageable **Modules** or **Sub-modules**. This prevents the AI from being overwhelmed by information and writing incomplete test cases.

## How to use

1. Ensure the questions from Step 2 have been fully answered.
2. Send the `prompt.txt` file to the AI.
3. The AI will return:
   - A list of **Modules / Sub-modules** with functional descriptions
   - **Dependencies** (dependency relationships between modules)
4. Quickly review the results → move to Step 4.

## Decomposition strategy

There are 3 ways to decompose, depending on the project:

| Approach      | Best when                              | Example                                               |
| ------------- | -------------------------------------- | ----------------------------------------------------- |
| **By UI**     | The page has clearly distinct sections | Header, Sidebar, Data Table, Form Popup               |
| **By flow**   | The feature has many CRUD operations   | Create flow, Edit flow, Delete flow                   |
| **By entity** | The system has many entities           | User management, Product management, Order management |

Customize the `[Hint]` section in prompt.txt so the AI decomposes in the most suitable way.
