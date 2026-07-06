---
name: generate-api-tests-from-swagger
description: Generate API test cases and automation scripts from a Swagger/OpenAPI specification. Supports 2 modes — SPEC (test cases only) and FULL (test cases + automation scripts).
---

# Workflow: Generate API Tests from Swagger/OpenAPI

> **MANDATORY SKILL:** You MUST load and carefully read the content of the **`qa-automation-engineer`** skill (at `.agents/skills/qa-automation-engineer/SKILL.md`) before starting. In addition, also refer to the **`test-data-generator`** skill to generate test data correctly.

This workflow helps the agent analyze a Swagger/OpenAPI specification, identify the endpoints, generate structured API test cases, and (depending on the mode) automatically generate complete automation scripts.

## ⚠️ Execution principles

- **All output in English**
- **Do NOT guess** the schema/endpoint — you must read the actual spec (JSON/YAML)
- **You must wait for user confirmation** of the scope at Step 2 before generating details
- If the user has not provided a Swagger URL/file → ask before starting
- ⚠️ **Rule E3:** When a test FAILS → read the log → analyze → fix → run again. Do NOT ask the user during the error-fixing process

## The 2 Modes

| Mode | When to use | Output |
|---|---|---|
| **SPEC** (default) | The user needs API test cases as a document | API Test Cases (Markdown) + Test Data Matrix |
| **FULL** | The user asks for automation scripts too | Same as SPEC + Automation Scripts + Project Structure |

> If the user says "generate automation", "write API test code", or asks for scripts → automatically switch to **FULL mode**.

## Steps

### Step 1: Receive & Analyze the Spec (Parse & Analyze)

1. **Collect the Swagger/OpenAPI spec** from the user:
   - **Direct URL** (e.g. `https://api.example.com/swagger.json`) → fetch the webpage
   - **Local file** (JSON/YAML) → read the file
   - **Swagger UI URL** → extract the original spec URL (usually `/v2/api-docs` or `/v3/api-docs`)
   - **Scalar API Reference URL** → inspect the HTML to find `data-configuration` containing the spec URL (usually `/swagger/json`, `/reference/json`, or a relative path in the `url` attribute). E.g. `https://example.com/swagger` → spec at `https://example.com/swagger/json`
   - **Other API Doc formats** (Redoc, Stoplight, RapiDoc) → find the spec URL in the page source or network requests
2. **Parse the spec** and extract the information:
   - Base URL, API version, authentication scheme (Bearer, API Key, OAuth2, Basic)
   - The list of all endpoints: `method + path`
   - Request parameters: path, query, header, body (schema + required fields)
   - Response schemas: status codes, response body structure
   - Models/Definitions: reusable data models
3. **Classify the endpoints** into groups:
   - **CRUD operations** — Create, Read, Update, Delete
   - **Authentication** — Login, Register, Token refresh
   - **Business Logic** — APIs handling complex business rules
   - **Utility** — Health check, config, metadata

### Step 2: Confirm Scope & Tech Stack (CHECKPOINT — ⏸️ STOP)

1. **Present a summary** for user review:
   - Total number of endpoints found (grouped)
   - Authentication method
   - The list of endpoint groups + the number of APIs in each group
   - The proposed mode (SPEC or FULL)
2. **Ask the user to confirm:**
   - "Do you want to test all endpoints or focus on a specific group?"
   - "Do you want the output as test cases (SPEC) or automation scripts too (FULL)?"
   - If FULL mode: "What tech stack do you want?" (default from the table below)
3. **Wait for the user to confirm** the scope before moving to Step 3

**Default Tech Stack (FULL mode):**

| Framework | Language | When to use |
|---|---|---|
| **REST Assured** | Java | Default for Java projects, TestNG runner |
| **Playwright API Testing** | TypeScript | When the user uses Playwright or a TypeScript stack |
| **Supertest + Jest** | TypeScript/JS | When the user uses a Node.js backend |
| **Requests + Pytest** | Python | When the user uses a Python stack |

### Step 3: Generate API Test Scenarios & Test Data

1. **For each endpoint** in the confirmed scope, generate test scenarios across 7 types:
   - **✅ Happy Path** — Valid request, response matches schema + status code
   - **❌ Negative — Validation** — Missing required fields, wrong data type, exceeds max length
   - **❌ Negative — Auth** — No token, expired token, token with the wrong role
   - **🔲 Boundary** — Min/max values, empty string, null, special characters
   - **⚡ Edge Cases** — Concurrent requests, duplicate creation, large payload, unicode/emoji
   - **🔒 Security** — SQL injection, XSS, IDOR, sensitive data exposure (see item 5 below)
   - **📄 Pagination & Filtering** — Pagination, sorting, search (see item 6 below)

2. **For each scenario**, define clearly:
   - **Request:** Method, URL, Headers, Body/Params (concrete values)
   - **Expected Response:** Status code, Response body structure, Error message
   - **Priority:** P1 (Critical) / P2 (High) / P3 (Medium) / P4 (Low)

3. **Generate the Test Data Matrix** (using the `test-data-generator` skill):
   - Valid data for the Happy Path
   - Invalid data for Negative cases (one negative set per field)
   - Boundary values per schema constraints (minLength, maxLength, min, max, pattern)
   - Data must be **unique + traceable** (e.g. `auto_api_1712049200@test.com`)

4. **Field-Level Validation for the Request Body (MANDATORY):**

   For each endpoint with a request body (POST/PUT/PATCH), the agent **MUST list each field** in the body and generate separate negative TCs for EACH field:

   | Field Type | Validation to test |
   |---|---|
   | **String (name, title...)** | Required → send missing · Empty string `""` · Whitespace only `"   "` · Exceeds maxLength · Below minLength · Special characters `<>&"'` · Unicode/Emoji · XSS payload `<script>alert(1)</script>` · SQL injection `' OR 1=1--` |
   | **Email** | Wrong format (missing `@`, missing domain) · Duplicate email (if unique) · Max length · Case sensitivity |
   | **Number (age, price...)** | String type instead of number · Negative number · Zero · Decimal (if integer) · Overflow (`999999999999`) · Min/Max value |
   | **Boolean** | Send string `"true"` instead of boolean `true` · Send `null` · Send number `1`/`0` |
   | **Date/DateTime** | Wrong format · Non-existent date (`2024-02-31`) · Future/past date (per rule) · Different timezones |
   | **Enum** | Value outside the enum · Case sensitivity · Empty string |
   | **Array** | Empty array `[]` · Too many elements · Element with wrong type · Duplicate elements |
   | **Object (nested)** | Missing required sub-fields · Extra fields not in the schema · Nested object `null` |
   | **File (multipart)** | Wrong MIME type · Exceeds size limit · Empty file (0 bytes) · File name with special characters |

   **Example — POST /api/customers `{ name, email, phone, age }`:**
   ```
   → Field "name" (string, required, maxLength: 100):
     TC: Send missing name field → 400 + error message
     TC: name = "" (empty) → 400
     TC: name = "   " (whitespace) → 400
     TC: name = 101 characters → 400 (exceeds max)
     TC: name = "<script>alert(1)</script>" → 400 or sanitized
   
   → Field "email" (string, required, format: email):
     TC: email = "invalid" → 400
     TC: email = "test@" → 400
     TC: email duplicate of another user → 409 Conflict
   
   → Field "age" (integer, min: 0, max: 150):
     TC: age = "abc" → 400 (wrong type)
     TC: age = -1 → 400 (below min)
     TC: age = 151 → 400 (exceeds max)
     TC: age = 18.5 → 400 (decimal instead of integer)
   ```

   > **Principle:** Each field has its own schema → its own validation TCs. Do NOT combine multiple fields into a single TC. Test each field independently first, then test combinations.

5. **API Security Testing:**

   Generate security test cases for each endpoint:

   | Type | Test Scenarios |
   |---|---|
   | **Injection** | SQL injection in query params (`?id=1 OR 1=1`) · SQL injection in body fields · XSS in input fields · Command injection (if the API runs shell) |
   | **IDOR** | Access another user's resource by ID (`GET /users/999` when the user may only view user 123) · Change the ID in PUT/DELETE to modify/delete a resource that is not theirs |
   | **Auth Bypass** | Call the API without a token → must be 401 · Expired token → 401 · Low-role token calling a high-role API → 403 · Tampered token (modified payload) → 401 |
   | **Sensitive Data** | Response does not return password/secret · Response does not leak internal IDs/stack traces · Headers do not leak server info (`X-Powered-By`, `Server`) |
   | **Rate Limiting** | Send many requests in a row → must be limited (429) · Brute force login → lock account |
   | **CORS** | Check the `Access-Control-Allow-Origin` header · Check the preflight request (OPTIONS) |

6. **Pagination & Filtering Tests (for GET List endpoints):**

   | Test Type | Scenarios |
   |---|---|
   | **Pagination** | No page/limit passed → use the default · `page=1&limit=10` → exactly 10 items · `page=999` → returns an empty array (no error) · `limit=0` or `limit=-1` → handled reasonably · `limit=10000` (too large) → limited or error |
   | **Sorting** | `sort=name&order=asc` → results in the correct order · `sort=invalid_field` → error or ignored · Case sensitivity in sort |
   | **Filtering** | Filter by each supported field · Filter with a non-existent value → returns an empty array · Filter with special characters · Combine multiple filters at once |
   | **Search** | Partial match search · Case-insensitive search · Search with special characters · Search with an empty string |

7. **Distinguish PUT vs PATCH (if the API has both):**

   | Method | Test Focus |
   |---|---|
   | **PUT** | Send all fields → full update · Send a missing optional field → that field is reset/null · Send a missing required field → 400 |
   | **PATCH** | Send only 1 field → only that field changes, the others stay the same · Send an empty body `{}` → nothing changes (or 400) · Send a non-existent field → ignore or 400 |

### Step 4: Package the API Test Cases (Output — SPEC mode)

1. Create the **artifact** `api_test_cases.md` with the structure:
   - **API Overview** — Base URL, Version, Auth method, Total endpoints
   - **Endpoint Catalog** — Table: `| # | Method | Path | Description | Number of Test Cases |`
   - **Detailed Test Cases** — Per endpoint:

   ```
   | TC ID | Endpoint | Scenario | Request | Expected Response | Priority | Type |
   ```

   - **Test Data Matrix** — Table of valid/invalid/boundary data for each model
   - **Dependencies & Execution Order** — Test run order (e.g. must create a user before testing get user)

2. If the user chooses **SPEC mode** → **STOP** here

### Step 5: Generate Automation Scripts (FULL mode)

> Only perform this when in **FULL mode**

1. **Design a project structure** appropriate for the framework:

   **REST Assured (Java):**
   ```
   src/test/java/
   ├── api/                    # API client classes (per resource)
   │   ├── BaseApi.java        # Base config: baseURI, auth, logging
   │   ├── UserApi.java        # Methods: createUser(), getUser(), ...
   │   └── AuthApi.java        # Methods: login(), refreshToken(), ...
   ├── models/                 # POJO/DTO classes (from Swagger models)
   │   ├── UserRequest.java
   │   └── UserResponse.java
   ├── tests/                  # Test classes (TestNG)
   │   ├── UserApiTest.java
   │   └── AuthApiTest.java
   ├── utils/                  # Helpers
   │   ├── TestDataGenerator.java
   │   └── AssertionHelper.java
   └── testdata/               # Test data files (JSON/YAML)
   ```

   **Playwright API (TypeScript):**
   ```
   tests/
   ├── api/
   │   ├── helpers/
   │   │   ├── base-api.ts     # Base request context, auth
   │   │   ├── user-api.ts     # API methods per resource
   │   │   └── test-data.ts    # Data generators
   │   ├── user.api.spec.ts    # Test file per resource
   │   └── auth.api.spec.ts
   └── fixtures/
       └── api-fixtures.ts     # Shared fixtures (auth tokens, etc.)
   ```

   **Requests + Pytest (Python):**
   ```
   tests/
   ├── api/
   │   ├── conftest.py          # Fixtures: base_url, auth_token, api_client
   │   ├── helpers/
   │   │   ├── base_api.py      # Base API client (requests.Session)
   │   │   ├── user_api.py      # API methods per resource
   │   │   └── test_data.py     # Data generators
   │   ├── models/
   │   │   ├── user_model.py    # Pydantic/dataclass models
   │   │   └── response_model.py
   │   ├── test_user_api.py     # Test file per resource
   │   └── test_auth_api.py
   └── pytest.ini               # Pytest config
   ```

   **Supertest + Jest (TypeScript):**
   ```
   tests/
   ├── api/
   │   ├── helpers/
   │   │   ├── base-api.ts      # Supertest agent config
   │   │   ├── user-api.ts      # API methods per resource
   │   │   └── test-data.ts     # Data generators
   │   ├── models/
   │   │   └── user.model.ts    # TypeScript interfaces
   │   ├── user.api.test.ts     # Test file per resource
   │   └── auth.api.test.ts
   ├── jest.config.ts            # Jest config
   └── setup.ts                  # Global setup (auth token)
   ```

2. **Generate the code** in this order:
   - **Base API class** — Config baseURL, default headers, auth interceptor, request/response logging
   - **Model/DTO classes** — From the Swagger definitions/schemas
   - **API client classes** — Methods for each endpoint (return typed response)
   - **Test Data generators** — Data factory with random + traceable values
   - **Test classes** — Call the API client, assert status code + response body + schema

3. **Mandatory assertions** for each test:
   - ✅ HTTP Status Code (exact match)
   - ✅ Response body — key fields matching expected values
   - ✅ Response time < threshold (if there is an SLA)
   - ✅ Response schema validation (correct structure)
   - ✅ Exact error message for negative cases
   - ✅ Headers check (Content-Type, CORS if relevant)

4. **Best practices in the code:**
   - Use a **Builder pattern or Factory** for test data
   - **Chain requests** when you need to set up data (e.g. create → get → update → delete)
   - **Soft assertions** when you need to check many fields at once
   - **Parameterized tests** for data-driven scenarios
   - **Cleanup/teardown** — delete the test data created after the test finishes
   - Do not hardcode token/credentials — read from env or config

### Step 6: Trial run & Auto-fix errors (Execution & Auto-Heal)

> Only perform this when in **FULL mode**

1. **Run the test** in the terminal:
   - REST Assured: `mvn test -Dtest=<TestClass>`
   - Playwright: `npx playwright test tests/api/`
   - Pytest: `python -m pytest tests/api/`

2. **Monitor** the result by checking the terminal output:
   - If **PASS** → update the artifact, report the result
   - If **FAIL** → apply the Auto-Heal loop:

   ```
   WHILE test FAIL:
     1. Read the error log → identify the root cause
     2. Classify the error:
        - Schema mismatch → update the model/assertion
        - Auth failure → check the token flow
        - 404/405 → recheck the endpoint path/method
        - Timeout → increase the timeout or check the server
        - Data conflict → use new unique test data
     3. Edit the code to apply the fix
     4. Run the test again
     5. Repeat until PASS (max 5 loops)
   ```

3. **⚠️ Rule E3:** Do NOT ask the user during the error-fixing process. Only ask when:
   - The API server does not respond (down/blocked)
   - A business rule conflicts with the spec
   - The 5 auto-heal loops are exhausted and it still fails

## Handling common errors

| Error | Cause | How to handle it |
|---|---|---|
| Swagger URL returns HTML | The URL is the Swagger UI, not the spec | Find the original spec URL (`/v2/api-docs`, `/v3/api-docs`, `/swagger.json`) |
| Scalar URL returns HTML | The URL is the Scalar API Reference UI | Inspect the HTML to find `data-configuration` → take the `url` field → join it with the base URL. Try: `/swagger/json`, `/reference/json` |
| Spec empty or cannot be parsed | Corrupt file or non-standard format | Ask the user to provide it again, try converting YAML ↔ JSON |
| Auth endpoint unclear | The spec does not document the auth flow | Ask the user about the auth method and how to obtain the token |
| Response schema differs from spec | The actual API does not match the document | Note it as a known issue, adjust the test to the actual response |
| Rate limiting | The API limits requests/minute | Add a delay between tests or use retry logic |
| CORS blocked | The browser blocks the cross-origin request | Check the `Access-Control-Allow-Origin` header, test directly with an HTTP client (not through the browser) |
| 405 Method Not Allowed | Calling the wrong HTTP method | Recheck the spec — confirm the correct method (PUT vs PATCH, POST vs PUT) |
| SSL/TLS Certificate Error | Self-signed or expired cert | Add an option to skip SSL verification in the test config (test environment only) |

## Output

### SPEC mode
- Artifact `api_test_cases.md`:
  - API Overview (Base URL, Version, Auth)
  - Endpoint Catalog (summary table)
  - Detailed Test Cases (grouped by endpoint)
  - Test Data Matrix
  - Execution Order & Dependencies

### FULL mode
- All output of SPEC mode, plus:
  - Project structure (appropriate for the framework)
  - Base API class + Auth helper
  - Model/DTO classes
  - API client classes (per resource)
  - Test classes with complete assertions
  - Test data generators
  - Test run result (PASS/FAIL report)