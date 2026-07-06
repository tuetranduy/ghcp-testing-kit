# Step 1: Initialize Context (Context & Role-play)

**Workflow:** generate-automation-from-testcases
**Skill:** qa-automation-engineer

---

## Purpose

Shape the AI into the role of a **Senior Automation Engineer** and load the technical context. This step lets the AI know exactly:
- Which framework (Playwright / Selenium / Appium)
- Which language (TypeScript / Java)
- What the project architecture looks like
- Which coding principles must be followed

## How to use

1. Open the `prompt.txt` file.
2. Replace the parts in `[...]`:
   - **Tech Stack:** Choose the framework, language, and build tool
   - **Goal:** The system/feature to automate
   - **Context:** Web architecture, frontend technology, element characteristics
   - **Architecture:** Copy from `project_architecture/README.md` or describe your existing project
3. Send it to the AI and wait for confirmation → move to Step 2.

## Notes

- Choose the right framework from the start — the AI will generate code in that framework's syntax throughout.
- If the project already exists, describe the current structure so the AI generates code in the correct directories.
- This step only needs to run **once** at the start of the conversation.
