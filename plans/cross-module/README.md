# AI-DRIVEN CROSS-MODULE TESTING FRAMEWORK

**Goal:**
Analyze and test complex features that traverse **multiple sequential modules**, where the output depends on a **multi-dimensional combination of conditions** (Combinatorial Testing).

## 📌 The Problem It Solves

When a feature **does not fit within a single module** but must pass through a chain of modules, each with multiple choices — and the combination of choices determines the final output (different templates, formulas, business rules).

**Real-world examples:**

| Feature                | Combination dimensions                                                |
| ---------------------- | --------------------------------------------------------------------- |
| Partner payment record | Partner type × Payment type × Tax × Debt × Asset source               |
| Insurance contract     | Insurance type × Subject × Package × Term × Payment method            |
| Export order           | Market × Goods type × Shipping × Payment × Documents                  |
| Approval process       | Request type × Department × Level × Amount → Different approval flows |

**If each dimension has 3-5 values →** the Full Cartesian combination easily reaches **hundreds of combinations**.

---

## 🚀 Two-Phase Process

### Phase 1: Analysis & Matrix Generation (generate-cross-module-test-plan)

| Step  | Name                    | Description                                                                 | Wait for user?   |
| ----- | ----------------------- | --------------------------------------------------------------------------- | ---------------- |
| **1** | Multi-Module Recon      | The AI opens the browser to explore each module, collecting fields + values | ❌                |
| **2** | Data Flow Mapping       | Determine what module A outputs → input to module B                         | ✅ **Checkpoint** |
| **3** | Dimension Extraction    | List all combination "dimensions" + values + constraints                    | ❌                |
| **4** | Combinatorial Matrix    | Generate the combination matrix (Pairwise / Business-critical / Full)       | ❌                |
| **5** | Expected Output Mapping | Map the expected template + formula for each combination                    | ✅ **Checkpoint** |

**Main output:** The combination matrix table — ready to import into Excel/Jira.

### Phase 2: Test Data Generation (generate-combinatorial-test-data)

| Mode         | When to use                                                   | Output                        |
| ------------ | ------------------------------------------------------------- | ----------------------------- |
| **GENERATE** | Generate data offline (JSON/CSV/Code)                         | Structured test data file     |
| **PIPELINE** | Run for real through the browser to create data on the system | Real data + IDs + screenshots |

---

## 3 Matrix Strategies

| Strategy               | Description                                         | When to use                            | Example                 |
| ---------------------- | --------------------------------------------------- | -------------------------------------- | ----------------------- |
| **Pairwise** (Default) | Cover 100% of pairs between any 2 dimensions        | Large combinations (>50)               | 216 combos → ~20 combos |
| **Business-Critical**  | Select only the most important combinations by risk | Need focus, limited time               | 216 combos → ~10 combos |
| **Full Cartesian**     | Test ALL valid combinations                         | Critical systems (finance, healthcare) | 216 combos → 216 combos |

> 💡 **Pairwise Testing** reduces the number of combinations by 80-90% while still catching the majority of defects.

---

## 🔗 Complete End-to-End Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                    CROSS-MODULE TESTING FLOW                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  📋 Step 0: Analyze Requirements per Module                     │
│      Skill: generate-requirements-from-website (run N times)    │
│                          ↓                                       │
│  📊 Step 1: Cross-Module Analysis & Matrix Generation           │
│      Skill: generate-cross-module-test-plan                     │
│      Output: Data Flow Map + combination matrix                 │
│                          ↓                                       │
│  🗃️ Step 2: Generate Test Data for the Matrix                   │
│      Skill: generate-combinatorial-test-data                    │
│      Output: Test data sets (offline or on the system)          │
│                          ↓                                       │
│  📝 Step 3: Generate Detailed Test Cases                        │
│      Skill: generate-manual-testcases-rbt (FULL RBT)            │
│      Input: Matrix + Requirements                               │
│      Output: Complete test cases                               │
│                          ↓                                       │
│  🤖 Step 4: Generate Automation Scripts                         │
│      Skill: generate-automation-from-testcases                  │
│      Output: Stable PASSing scripts                            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📁 Directory Structure

```
plans/cross-module/
├── README.md              ← Introduction + Overview (the file you are reading)
└── QUICK_START.md         ← Quick usage guide (sample prompts + run flow)
```

**Referenced skills:**

```
.agents/skills/
├── generate-cross-module-test-plan/SKILL.md       ← Analysis + Matrix
└── generate-combinatorial-test-data/SKILL.md      ← Test data generation
```

**Extended skill:**

```
.agents/skills/test-data-generator/SKILL.md   ← Multi-Step Pipeline + Combinatorial Data
```

---

## 📋 Quick guide

See the `QUICK_START.md` file in this directory to get started quickly.
