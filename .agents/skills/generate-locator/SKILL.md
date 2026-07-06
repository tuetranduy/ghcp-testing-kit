---
name: generate-locator
description: Generate stable locators for UI elements. Supports Playwright, Selenium, Appium.
---

# generate-locator — Generate Stable Locators for UI Automation

> The user provides the element to find a locator for (a description, screenshot, URL, or HTML snippet).
> The AI inspects the real DOM/UI hierarchy, generates a stable locator following the standard priority, verifies uniqueness, and returns the result.

> **MANDATORY:** Before starting, you MUST load and carefully read:
> - **Skill:** `.agents/skills/smart-locator-agent/SKILL.md` — The locator generation process
> - **Skill:** `.agents/skills/ui-debug-agent/SKILL.md` — The DOM inspection process
> - **Rule:** `.agents/rules/locator_strategy.md` — The locator priority map
> - **Rule:** `.agents/rules/<framework>_rules.md` — Rules specific to the framework in use

---

## Inputs required from the User

| Input | Required | Description |
|-------|----------|-------|
| Element description | ✅ | E.g.: "Login button", "Country dropdown", "Email input field" |
| URL of the page containing the element | ✅ | So the AI can navigate and inspect the real DOM |
| Framework | ✅ | `playwright`, `selenium`, or `appium` |
| HTML snippet | ❌ | If the User already has DOM context — the AI uses it for quick analysis |
| Target Page class | ❌ | The Page class file the locator will be added to |
| Login required | ❌ | If the page requires login — the User specifies how to log in |

> **Note on Login:** If the page requires login, the User MUST specify how to log in (fixture, script, login URL...). The AI MUST NOT read `.env` or guess credentials.

---

## Steps

### Phase 1: Analyze the request

1. **Understand the element to find** — Clearly determine:
   - **Element type:** button, input, link, dropdown, dialog, table row, checkbox, radio...
   - **Context:** Is it in the main page, a dialog/modal, sidebar, table, iframe?
   - **Action:** click, fill, select, hover, verify text, verify visibility?

2. **Determine the framework and read the corresponding rules:**

   | Framework | Rule file |
   |-----------|-----------|
   | Playwright | `.agents/rules/playwright_rules.md` |
   | Selenium | `.agents/rules/selenium_rules.md` |
   | Appium | `.agents/rules/appium_rules.md` |

3. **Check the current Page class (if the User specifies it):**
   - Read the Page class file → know which locators already exist
   - Avoid duplication or naming conflicts

---

### Phase 2: Inspect the real DOM / UI Hierarchy

> ⚠️ **AN IMMUTABLE PRINCIPLE: NEVER GUESS A LOCATOR. YOU MUST INSPECT THE REAL THING.**

#### 2A. Web (Playwright MCP)

4. **Navigate to the page containing the element:**
   ```
   browser_navigate(url=<target_url>)
   ```

5. **Resize the viewport (MANDATORY):**
   ```
   browser_resize(width=1920, height=1080)
   ```

6. **Capture the DOM:**
   ```
   browser_snapshot()
   ```

7. **Analyze the element in the snapshot:**
   - Find the element ref in the DOM tree
   - Record **all attributes that have a value**: `role`, `aria-label`, `aria-labelledby`, `data-testid`, `data-test`, `data-qa`, `id`, `name`, `placeholder`, `type`, `href`, text content
   - Record the **parent context** (dialog? table? sidebar? iframe?)

8. **If the element is hidden** (dropdown menu, modal, tooltip...):
   - Perform the action to open the element: `browser_click(ref=<trigger>)`
   - Capture again: `browser_snapshot()`

#### 2B. Mobile (Appium)

4. **Use Appium Inspector** or `page_source` to get the UI hierarchy
5. **Record the attributes:** `accessibility-id`, `resource-id`, `content-desc`, `text`, `class`, `bounds`
6. **If the element is inside a scroll view** → scroll to the element before inspecting

---

### Phase 3: Generate the locator by Priority

9. **Apply the Master Priority Map:**

   | # | Type | When to use |
   |---|------|-------------|
   | 1 | Accessibility / Aria | Element has a clear `role`, `aria-label` |
   | 2 | Test attribute | Element has `data-testid`, `data-test`, `data-qa` |
   | 3 | ID / name | Element has a stable `id` or `name` (not auto-generated) |
   | 4 | Framework semantic | Playwright: `getByRole`, `getByLabel`... |
   | 5 | CSS Selector | Use a specific attribute, avoid dynamic classes |
   | 6 | XPath | Last resort — use relative XPath only |

10. **Generate the locator per framework:**

    **Playwright (TypeScript/JavaScript):**
    ```typescript
    // Priority 1: Role-based
    page.getByRole('button', { name: 'Submit' })

    // Priority 2: Test ID
    page.getByTestId('submit-btn')

    // Priority 3: Label / Placeholder
    page.getByLabel('Email')
    page.getByPlaceholder('Enter your password')

    // Priority 4: Text
    page.getByText('Submit')

    // Priority 5: CSS
    page.locator('#submit-button')
    page.locator('[data-testid="submit-btn"]')

    // Priority 6: XPath (last resort)
    page.locator('//button[@type="submit"]')
    ```

    **Playwright (Python):**
    ```python
    # Priority 1: Role-based
    page.get_by_role("button", name="Submit")

    # Priority 2: Test ID
    page.get_by_test_id("submit-btn")

    # Priority 3: Label / Placeholder
    page.get_by_label("Email")
    page.get_by_placeholder("Enter your password")

    # Priority 4: Text
    page.get_by_text("Submit")

    # Priority 5: CSS
    page.locator("#submit-button")

    # Priority 6: XPath (last resort)
    page.locator("//button[@type='submit']")
    ```

    **Selenium (Java):**
    ```java
    // Priority 1: ID
    driver.findElement(By.id("submit-button"));

    // Priority 2: Test attribute
    driver.findElement(By.cssSelector("[data-testid='submit-btn']"));

    // Priority 3: Name
    driver.findElement(By.name("submit"));

    // Priority 4: CSS Selector
    driver.findElement(By.cssSelector("button.btn-primary[type='submit']"));

    // Priority 5: XPath (last resort)
    driver.findElement(By.xpath("//button[@type='submit']"));
    ```

    **Appium (Java):**
    ```java
    // Priority 1: Accessibility ID
    driver.findElement(AppiumBy.accessibilityId("login_button"));

    // Priority 2: Resource ID (Android)
    driver.findElement(AppiumBy.id("com.app:id/login_button"));

    // Priority 3: iOS Predicate
    driver.findElement(AppiumBy.iOSNsPredicateString("label == 'Login'"));

    // Priority 4: Class Chain (iOS)
    driver.findElement(AppiumBy.iOSClassChain("**/XCUIElementTypeButton[`label == 'Login'`]"));

    // Priority 5: XPath (last resort)
    driver.findElement(AppiumBy.xpath("//android.widget.Button[@text='Login']"));
    ```

---

### Phase 4: Validate the locator

11. **Verify uniqueness — it MUST match exactly 1 element:**

    **Web (Playwright MCP):**
    ```
    browser_evaluate(function="() => document.querySelectorAll('<css_selector>').length")
    ```

    **Selenium:**
    ```java
    List<WebElement> matches = driver.findElements(By.<strategy>("<locator>"));
    assert matches.size() == 1;
    ```

    **Appium:**
    ```java
    List<WebElement> matches = driver.findElements(AppiumBy.<strategy>("<locator>"));
    assert matches.size() == 1;
    ```

12. **Verify visibility** — the element must be interactable:
    - Not overlaid by another element
    - Not in a `hidden`, `display:none`, `visibility:hidden` state
    - Not outside the viewport (requiring a scroll)

13. **Verify stability — Check the checklist:**
    - [ ] Does not use dynamic CSS classes (e.g.: `css-1n2xyz`, `sc-bdnxRM`)
    - [ ] Does not use absolute XPath (e.g.: `//html/body/div[1]/div[2]/button`)
    - [ ] Does not use auto-generated IDs (e.g.: `ember123`, `react-select-2-input`)
    - [ ] Does not use `nth-child` / `nth-of-type` when a better option exists
    - [ ] Survives a page reload
    - [ ] Stable across multiple page states (loading, loaded, with data, without data)

---

### Phase 5: Return the result

14. **Output Format — you MUST provide all 3 sections:**

```markdown
## Locator Result: [Element description]

**Framework:** [Playwright / Selenium / Appium]

### 🎯 Primary Locator (Recommended)
```<language>
// Locator code — copy-paste ready
```
- **Type:** [Role-based / Test ID / CSS / ...]
- **Unique:** ✅ Matches 1 element
- **Stability:** ✅ Does not use dynamic class / absolute xpath

### 🔄 Fallback Locator
```<language>
// Alternative locator for when the primary breaks
```
- **Type:** [CSS / XPath / ...]
- **When to use:** When the primary locator breaks due to a DOM change

### 💡 Reasoning
- Explain why this Primary locator was chosen
- Why other candidates were rejected
- Potential risks (if any)

### 📋 Usage Example (if the User requests it)
```<language>
// Example of using the locator in test code
```
```

15. **(Optional) If the User requests adding it to the Page class:**
    - Add the locator in the correct place in the Page class
    - Name it per the project's naming convention
    - Create a method that uses the locator if needed

---

## Common Patterns (Reference)

### Scoping a locator within a Dialog / Modal:
```typescript
// Playwright — scope into the dialog first
const dialog = page.getByRole('dialog');
dialog.getByRole('button', { name: 'Confirm' }).click();
```
```java
// Selenium — scope into the dialog
WebElement dialog = driver.findElement(By.cssSelector("[role='dialog']"));
dialog.findElement(By.cssSelector("button[data-testid='confirm']")).click();
```

### Dynamic text matching:
```typescript
// Playwright — exact vs partial
page.getByText('Submit', { exact: true })     // exact match
page.getByText(/submit/i)                      // regex, case-insensitive
```
```python
# Playwright Python — normalize-space XPath
page.locator(f"//a[normalize-space()='{text}']")
```

### Table row action:
```typescript
// Playwright — filter the row then interact
const row = page.getByRole('row').filter({ hasText: 'John Doe' });
row.getByRole('button', { name: 'Edit' }).click();
```
```java
// Selenium — relative XPath within a table
driver.findElement(By.xpath("//tr[contains(., 'John Doe')]//button[text()='Edit']"));
```

---

## PROHIBITED

| ❌ Do not do | ✅ Correct replacement |
|-------------------|-----------------|
| Guess a locator without inspecting the DOM/UI | `browser_snapshot()` or Appium Inspector first |
| Use dynamic CSS classes (`css-1n2xyz`, `sc-xxx`) | Use role, aria, data-testid, text |
| Use absolute XPath (`//html/body/div[1]...`) | Use relative XPath with an attribute |
| Use auto-generated IDs (`ember123`, `:r1:`) | Use a stable attribute or text |
| Return a locator without verifying uniqueness | Always verify it matches exactly 1 element |
| Return only 1 locator with no fallback | Return Primary + Fallback + Reasoning |
| Read `.env` to obtain login credentials | Ask the User how to log in or use an existing fixture |

---

## Final checklist

- [ ] Inspected the real DOM/UI hierarchy (did not guess)
- [ ] Locator follows the priority: accessibility > test-id > id/name > semantic > css > xpath
- [ ] The primary locator matches exactly 1 element
- [ ] There is a Fallback locator
- [ ] There is Reasoning explaining the choice
- [ ] Does not use dynamic classes, absolute xpath, or auto-generated IDs
- [ ] The locator is stable across page reloads and multiple states
- [ ] (If added to a Page class) Verified the code runs without errors