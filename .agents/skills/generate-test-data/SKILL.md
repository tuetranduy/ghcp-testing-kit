---
name: generate-test-data
description: Generate structured, unique, and traceable test data for test cases. Supports UI forms, API payloads, and data-driven tests.
---

# generate-test-data — Generate Structured Test Data

> The user provides a feature/module or test cases that need data.
> The AI analyzes the fields and constraints, then generates a complete data set (positive, negative, boundary, edge cases) in a ready-to-use format.

> **MANDATORY:** Before starting, you MUST load and carefully read:
> - **Skill:** `.agents/skills/test-data-generator/SKILL.md` — Data generation rules
> - **Rule:** `.agents/rules/automation_rules.md` — Test Data section

---

## Input Required from the User

| Input | Required | Description |
|-------|----------|-------------|
| Feature / Module | ✅ | E.g., "Registration form", "API Create User", "Login page" |
| Fields needing data | ⚠️ Recommended | List of fields + constraints. If missing → the AI analyzes from the DOM/Spec |
| Page URL / Swagger spec | ❌ | If provided → the AI inspects the DOM/Spec to get accurate validation rules |
| Test cases | ❌ | If test cases already exist → the AI generates data matching each TC |
| Output format | ❌ | `json` (default), `csv`, `markdown table`, `code` (TypeScript/Java/Python) |
| Data language | ❌ | English / other (default: follow context) |

---

## Steps

### Step 1: Analyze Fields & Constraints

1. **Determine the information source:**

   | Source | How to obtain | Priority |
   |--------|---------------|----------|
   | Provided directly by the user | Read from input | ⭐ Highest |
   | Actual DOM (UI form) | `browser_navigate` → `browser_snapshot` → analyze input fields | ⭐ High |
   | Swagger/OpenAPI spec | Fetch the webpage → parse schema + constraints | ⭐ High |
   | Existing test cases | Read test steps → extract fields | Medium |
   | Guess from module name | Based on domain experience | ⭐ Lowest |

2. **For each field, determine:**

   | Attribute | Example |
   |-----------|---------|
   | **Field name** | email, password, name, phone |
   | **Data type** | string, number, boolean, date, file, enum |
   | **Required?** | Mandatory or optional |
   | **Validation rules** | min/maxLength, pattern/regex, format (email, phone), unique |
   | **Default value** | If any |
   | **Enum values** | E.g., status = ["AVAILABLE", "UNAVAILABLE"] |
   | **Dependency relationship** | E.g., `password_confirm` must match `password` |

3. **Present the Fields table for the user to confirm (CHECKPOINT ⏸️):**

   ```markdown
   | # | Field | Type | Required | Constraints | Notes |
   |---|-------|------|----------|-------------|-------|
   | 1 | email | string | ✅ | format: email, unique | Used to log in |
   | 2 | password | string | ✅ | minLength: 6 | |
   | 3 | name | string | ✅ | maxLength: 100 | |
   | 4 | phone | string | ❌ | pattern: /^0[0-9]{9}$/ | 10 digits, starts with 0 |
   ```

   > Wait for the user to confirm or add constraints before moving to Step 2.

---

### Step 2: Generate Test Data across 4 Categories

For each identified field, generate data according to the table below:

#### 2A. Positive Data (Happy Path)

- Valid values, correct format, within constraints
- All required fields are filled
- Data must be **unique + traceable**

**Unique data format:**
```
<prefix>_<testName>_<timestamp>
```
Examples:
```
email:    auto_register_1712049200@test.com
username: user_login_1712049200
code:     TC_BOOK_1712049200
```

#### 2B. Negative Data

| Type | Description | Example |
|------|-------------|---------|
| **Missing required** | Leave a required field empty | `email: ""` |
| **Invalid format** | Wrong format | `email: "not-an-email"` |
| **Invalid type** | Wrong data type | `price: "abc"` (expects number) |
| **Duplicate** | Value already exists | `email: "existing@test.com"` |
| **Invalid characters** | Disallowed characters | `name: "<script>alert(1)</script>"` |
| **Wrong relationship** | Violates field relationship | `password_confirm ≠ password` |

#### 2C. Boundary Values

| Type | Description | Example (field with minLength=6, maxLength=20) |
|------|-------------|-------|
| **Min** | Exactly at min | `"abcdef"` (6 chars) |
| **Min - 1** | Below min | `"abcde"` (5 chars) |
| **Max** | Exactly at max | `"a" * 20` (20 chars) |
| **Max + 1** | Above max | `"a" * 21` (21 chars) |
| **Empty** | Empty string | `""` |
| **Zero** | The number 0 (for number fields) | `0` |
| **Negative** | Negative number (for number fields) | `-1` |

#### 2D. Edge Cases

| Type | Description | Example |
|------|-------------|---------|
| **Unicode** | Special Unicode characters | `"Zoë Müller 🎉"` |
| **Very long** | Extremely long string | `"a" * 10000` |
| **Whitespace** | Leading/trailing spaces | `"  email@test.com  "` |
| **SQL injection** | SQL injection pattern | `"'; DROP TABLE users; --"` |
| **HTML tags** | HTML in a text field | `"<b>bold</b><img src=x onerror=alert(1)>"` |
| **Null/undefined** | Null value | `null` |
| **Special numbers** | Special numbers | `0.1 + 0.2`, `Number.MAX_SAFE_INTEGER`, `NaN` |
| **Date edge** | Special dates | `"2024-02-29"` (leap year), `"2024-12-31"`, `"1970-01-01"` |

---

### Step 3: Package the Output

Return the result in the format requested by the user (default: JSON):

#### JSON Format (default)

```json
{
  "module": "User Registration",
  "totalDataSets": 15,
  "fields": ["email", "password", "name", "phone"],
  "positive": [
    {
      "id": "POS_01",
      "description": "Successful registration with all valid fields",
      "data": {
        "email": "auto_register_1712049200@test.com",
        "password": "Test@12345",
        "name": "Auto User Register",
        "phone": "0912345001"
      },
      "expectedResult": "Account created successfully, status 201"
    }
  ],
  "negative": [
    {
      "id": "NEG_01",
      "description": "Empty email",
      "data": { "email": "", "password": "Test@12345", "name": "Test User" },
      "expectedResult": "Validation error: Email is required",
      "targetField": "email",
      "negativeType": "missing_required"
    },
    {
      "id": "NEG_02",
      "description": "Invalid email format",
      "data": { "email": "not-email", "password": "Test@12345", "name": "Test User" },
      "expectedResult": "Validation error: Invalid email format",
      "targetField": "email",
      "negativeType": "invalid_format"
    }
  ],
  "boundary": [
    {
      "id": "BND_01",
      "description": "Password at exactly min length (6 chars)",
      "data": { "email": "auto_bnd01_1712049200@test.com", "password": "Abc@12", "name": "Test User" },
      "expectedResult": "Success",
      "targetField": "password",
      "boundaryType": "min"
    }
  ],
  "edgeCases": [
    {
      "id": "EDGE_01",
      "description": "Name contains non-ASCII Unicode + emoji",
      "data": { "email": "auto_edge01_1712049200@test.com", "password": "Test@12345", "name": "Zoë Müller 🎉" },
      "expectedResult": "Success — the system accepts Unicode",
      "targetField": "name",
      "edgeType": "unicode"
    }
  ]
}
```

#### Markdown Table Format

```markdown
| ID | Category | Description | email | password | name | Expected Result |
|----|----------|-------------|-------|----------|------|-----------------|
| POS_01 | Positive | Successful registration | auto_reg@test.com | Test@12345 | Auto User | 201 Created |
| NEG_01 | Negative | Empty email | (empty) | Test@12345 | Test User | 422: Email is required |
| BND_01 | Boundary | Password min length | auto_bnd@test.com | Abc@12 | Test User | 201 Created |
```

#### Code Format (TypeScript example)

```typescript
// test-data/registration.data.ts
export const registrationData = {
  positive: {
    email: `auto_register_${Date.now()}@test.com`,
    password: 'Test@12345',
    name: 'Auto User Register',
    phone: '0912345001',
  },
  negative: {
    emptyEmail: { email: '', password: 'Test@12345', name: 'Test' },
    invalidEmail: { email: 'not-email', password: 'Test@12345', name: 'Test' },
    shortPassword: { email: `auto_neg_${Date.now()}@test.com`, password: '123', name: 'Test' },
  },
  boundary: {
    minPassword: { email: `auto_bnd_${Date.now()}@test.com`, password: 'Abc@12', name: 'Test' },
    maxName: { email: `auto_bnd_${Date.now()}@test.com`, password: 'Test@12345', name: 'A'.repeat(100) },
  },
};
```

---

## Data Rules (MANDATORY)

| # | Rule | Description |
|---|------|-------------|
| 1 | **Unique** | No duplicates within the test suite — use timestamp/random |
| 2 | **Traceable** | Able to trace which test generated the data — use prefix + test name |
| 3 | **No real PII** | Do NOT use real personal data (real ID numbers, real emails, real phone numbers) |
| 4 | **Respect constraints** | Data must comply with the analyzed validation rules |
| 5 | **Include expectedResult** | Each data set MUST have an expected result for assertions |
| 6 | **Deterministic when needed** | Same seed → same data (for reproducible tests) |

---

## PROHIBITED

| ❌ Do NOT | ✅ Correct alternative |
|-----------|------------------------| 
| Use placeholders (`valid email`, `valid code`) | Concrete values: `auto_tc01@test.com`, `KH-2026-0012` |
| Hardcode duplicate data across tests | Random data with prefix + timestamp |
| Use real personal data | Fake data: `auto_*@test.com` |
| Generate only positive data | All 4 categories required: Positive + Negative + Boundary + Edge |
| Generate data with no expected result | Each data set MUST state its expected result clearly |
| Guess validation rules without verifying | Inspect the DOM/Spec or ask the user |
| Read `.env` to get credentials | Ask the user or use the placeholder `[FROM_ENV]` |

---

## Final Checklist

- [ ] Fields + constraints have been fully analyzed
- [ ] The user confirmed the fields table before data was generated
- [ ] Data covers all 4 categories (Positive, Negative, Boundary, Edge Cases)
- [ ] Each data set has a clear expected result
- [ ] Data is unique + traceable (prefix + timestamp/random)
- [ ] No real PII
- [ ] Output format matches the user's request (JSON/CSV/Markdown/Code)
- [ ] Boundary values match the constraints (min, min-1, max, max+1)