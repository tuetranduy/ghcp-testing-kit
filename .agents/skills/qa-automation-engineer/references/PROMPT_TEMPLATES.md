# Prompt Templates

Reusable prompt templates for common QA automation tasks. The agent can refer to these when it needs to format output or when the user's request is not yet clear.

---

## 1. Test Case Generation

```
Analyze the following requirement and generate test cases:

**Requirement:** [Requirement description]

**Output format:**
| TC ID | Test Case Title | Precondition | Steps | Expected Result | Priority | Type |

**Requirements:**
- Include positive, negative, boundary, and edge cases
- Use English for descriptions
- Priority: High / Medium / Low
- Type: Positive / Negative / Boundary / Edge
```

---

## 2. Automation Script Generation

```
Convert the following test case into an automation script:

**Test Case:** [TC content]
**Framework:** [Selenium Java / Playwright TypeScript]
**Pattern:** Page Object Model

**Output:**
1. Page Object class(es)
2. Test class
3. Test data (if needed)

**Rules:**
- Smart waits only (no hard sleep)
- Random test data with prefix + timestamp
- Clear assertions
```

---

## 3. API Test Generation

```
Generate API tests from a Swagger specification:

**Swagger URL:** [URL]
**Endpoint(s):** [Endpoints to test]
**Framework:** REST Assured + TestNG

**Include:**
- Happy path (200 OK)
- Validation errors (400)
- Authentication (401/403)
- Not found (404)
- Boundary values
- Schema validation
```

---

## 4. Locator Generation

```
Inspect the element and generate a stable locator:

**Element:** [Description of the element to find]
**Page URL:** [URL]
**Tool:** [Selenium / Playwright]

**Output:**
- Primary locator
- Fallback locator
- Explanation of why the locator was chosen
```

---

## 5. Flaky Test Analysis

```
Analyze a flaky test and suggest a fix:

**Test file:** [Path to test]
**Symptoms:** [Description of the flaky behavior]

**Analysis:**
1. Root cause
2. Detected pattern (timing, data, environment, selector)
3. Specific fix suggestion
4. Code fix
```

---

## 6. Test Data Generation

```
Generate test data for a module:

**Module:** [Module name]
**Fields:** [List of fields needing data]

**Include:**
- Valid data (happy path)
- Invalid data (negative)
- Boundary values (min, max, empty, null)
- Special characters
- Format: JSON / CSV / Excel
```
