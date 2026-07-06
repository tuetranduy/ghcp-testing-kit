# 📋 Quick Start: Using the 6-Step AI-RBT Process

## 🔀 Choose Your Workflow

There are **2 distinct workflows**, depending on the AI tool you are using:

### Workflow 1: Agent skill — Automated

```
Use the generate-manual-testcases-rbt skill + paste requirements
→ The AI runs all 6 steps per the skill, pausing at checkpoints for you
→ No need to copy-paste prompt templates
```

**Pros:** Fast, automated, the agent keeps context throughout.
**Cons:** Guidance is at a general level (less detailed than the prompt templates).

### Workflow 2: Copy-Paste Prompts — Manual (any AI chat)

```
Copy the Step 1 prompt → paste into chat → AI processes it
→ Copy the Step 2 prompt → paste → AI processes it
→ ... repeat through Step 6
```

**Pros:** More detailed prompts, with concrete examples and deeper hints.
**Cons:** You must copy-paste manually 6 times.

---

## Workflow 1: Agent skill — Quick prompt

```
Use the generate-manual-testcases-rbt skill.

Project: [Project name]
Feature: [Feature name]
Goal: [Short description]

[Paste requirements/user stories here]
```

When the AI pauses at a checkpoint, simply answer the question or type:
```
Continue to Step [X]
```

---

## Workflow 2: Copy-Paste — Step-by-step guide

| Step  | Name                | Prompt file                                                              | Wait for user?                 |
| ----- | ------------------- | ------------------------------------------------------------------------ | ------------------------------ |
| **1** | Context & Role-play | Copy `plans/manual/01_context_and_roleplay/prompt.txt` + fill in `[...]` | ✅ Wait for confirmation        |
| **2** | Analysis & QnA      | Copy `plans/manual/02_analysis_and_qna/prompt.txt`                       | ✅ **Wait for Q&A answers**     |
| **3** | Decomposition       | Copy `plans/manual/03_decomposition/prompt.txt`                          | Quick review                   |
| **4** | Traceability        | Copy `plans/manual/04_traceability/prompt.txt`                           | ✅ **Wait for scenario review** |
| **5** | RBT & TC Generation | Copy `plans/manual/05_rbt_and_tc_generation/prompt.txt`                  | Review results                 |
| **6** | Template Mapping    | Copy `plans/manual/06_template_mapping/prompt.txt`                       | Copy table → Excel             |

### Flow diagram:

```
[Step 1] Copy prompt + paste requirements document
    ↓  AI confirms understanding → User confirms OK
[Step 2] Copy analysis prompt
    ↓  AI raises questions → ⏸️ User answers each one
[Step 3] Copy decomposition prompt
    ↓  AI generates module list → User quick review
[Step 4] Copy traceability prompt
    ↓  AI generates scenarios → ⏸️ User reviews + adds
[Step 5] Copy TC generation prompt
    ↓  AI generates detailed test cases → User review
[Step 6] Copy standardization prompt
    ↓  AI generates Markdown table → Copy into Excel/Jira ✅
```

---

## Optimization Tips

1. **Step 2 is the most important** — Do not rush; answer each question the AI raises carefully.
2. **Split modules when there are many** — At Step 5, if there are more than 5 modules, ask the AI to generate one module at a time.
3. **Review before formatting** — At Step 5, review the test cases before moving to Step 6.
4. **Use the same conversation** — Run all 6 steps in **the same conversation** so the AI retains context.
5. **The Copy-Paste workflow is more detailed** — If you need the highest quality, use Workflow 2 (even when using an agent skill).
