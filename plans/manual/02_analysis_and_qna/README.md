# Step 2: Requirement Analysis and Q&A

---

## Purpose

Ask the AI to analyze the documents from Step 1 to find gaps, contradictions, and ambiguities **before** writing scenarios. This step mirrors the Tester's mindset of reading the documents and asking questions of the BA/PO.

## How to use

1. Copy the content of `prompt.txt` and send it right after the AI has confirmed in Step 1.
2. The AI will return:
   - A list of **flows** (Happy / Alternate / Exception Paths)
   - A list of **Ambiguities** (detected unclear points)
   - A numbered list of **Q&A questions**
3. **Read each question carefully** and answer them for the AI.
4. Once all are resolved → move to Step 3.

## ⚠️ Important Note

- **This is the most important step** in the process. If skipped, the AI will guess the logic → seriously incorrect test cases.
- Answer **as specifically as possible**. If you are unsure, state clearly "Undetermined, assume that..." so the AI records it.
- You can add extra hints in the `[...]` sections of prompt.txt to steer the AI toward the area you care about.

