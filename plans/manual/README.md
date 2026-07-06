# AI-DRIVEN RISK-BASED TESTING FRAMEWORK (AI-RBT)

**Goal:**
Leverage the speed of AI for detailed execution, combined with human strategic Risk-Based Testing (RBT) thinking to optimize testing resources.

## 📌 Core Principles

1. **Human Strategy:** Humans define the strategy, risk levels, and standards.
2. **AI Execution:** AI performs analysis, writes test cases, and reviews gaps.
3. **Human Verification:** Humans review the AI's output before finalizing.

---

## 🚀 Process Overview (6 Steps)

1. **Context & Role-play:** Establish a QA/Tester expert mindset for the AI by shaping its role and providing project context.
2. **Analysis & QnA (Requirement Analysis):** The AI reads the documents and analyzes the requirements to clarify ambiguities before writing scenarios.
3. **Decomposition:** Break the system down into smaller Modules (Feature Mapping - FM) for easier assessment.
4. **Traceability (Ensuring Coverage):** Establish a traceability matrix to guarantee requirement coverage.
   - *(Note: After Step 4 there may be a human-only Risk Assessment checkpoint.)*
5. **RBT & TC Generation (Detailed Test Case Generation):** Apply the Risk-Based Testing strategy so the AI generates the test scenario content (Logic & Scenario).
6. **Template Mapping (Format Standardization):** Standardize the entire Test Case format and fill it into a template file/table for management use (for example, Jira, Excel).

*(Each step above corresponds to one subfolder in this directory, containing a detailed `README.md` guide and a `prompt.txt` with a sample prompt for the AI.)*

---

## ⚠️ Important Note: Execution Strategy

For **Manual Testing** (starting from functional requirements / Figma), you **MUST run each prompt sequentially by hand** instead of combining them into a single fully automated workflow.

**Reasons:**
1. **Bottleneck at Step 2 (Analysis & QnA):** The AI needs time to analyze and raise questions (ambiguities) about the requirement logic for you to answer. Running everything at once forces the AI to guess the logic, resulting in seriously incorrect test cases.
2. **Human in the loop:** At Steps 4 and 5, the human tester must personally review the risk assessment (RBT) before letting the AI generate detailed scenarios.
3. **Preventing hallucination:** Feeding in one long document and forcing the AI to produce hundreds of scenarios at once causes it to lose focus and miss coverage. Splitting the work into individual prompts yields the highest possible Test Case quality.
