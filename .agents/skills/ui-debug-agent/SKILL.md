---
name: ui-debug-agent
description: A skill for inspecting web/mobile applications with browser tools, analyzing DOM elements, identifying stable locators, debugging UI automation failures, and helping generate Page Object classes.
---

# UI Debug Agent

## Description

A specialized skill that helps the agent inspect web/mobile applications directly in a real browser, analyze the DOM, collect stable locators, and debug UI automation issues.

The agent can:

- Open a real browser and navigate to any URL
- Inspect DOM elements — identify attributes, hierarchy, state
- Collect stable locators for Playwright, Selenium, Appium
- Debug automation failures (element not found, click intercepted, timeout)
- Capture UI state (snapshot, screenshot) for analysis
- Analyze dynamic content, iframes, shadow DOM, SPA navigation

---

## When to Use

Use this skill when:

- You need to **explore the UI** of a new web page/module
- You need to **find a locator** for a specific element
- You need to **debug** a test automation failure caused by a UI change
- You need to **verify** whether a locator works against the real DOM
- You need to **analyze the DOM** to understand the UI structure (forms, tables, modals)
- You need to **capture evidence** (screenshots) for a test report

Trigger keywords: "inspect UI", "find locator", "debug element", "open the browser to look", "check the DOM"

---

## MCP Command Sequence (MANDATORY)

When using Playwright MCP to debug the UI, **ALWAYS** follow this order:

```
1. browser_navigate(url)           → Open the page
2. browser_resize(1920, 1080)      → Desktop viewport
3. browser_wait_for(text/time)     → Wait for the page to load
4. browser_snapshot()              → Collect the DOM (used for analysis + finding locators)
5. browser_click/type/hover(ref)   → Interact (if needed)
6. browser_take_screenshot()       → Capture image (evidence on failure or milestone)
```

### Important rules:

| Rule | Detail |
|---|---|
| **Do NOT re-navigate** if already on the correct page | Avoid unintended reloads |
| **ALWAYS resize** right after navigation | `browser_resize(1920, 1080)` — ensures a desktop viewport |
| **ALWAYS wait** before snapshotting | Wait for the page to finish loading |
| **Use snapshot for analysis** | The snapshot returns the accessibility tree — fast, accurate, with a `ref` to interact |
| **Use screenshot for reporting** | The screenshot is an image — use it when visual evidence is needed |

---

## Snapshot vs Screenshot

| | `browser_snapshot` | `browser_take_screenshot` |
|---|---|---|
| **Returns** | Accessibility tree (text + ref IDs) | Image (PNG/JPEG) |
| **Purpose** | Analyze the DOM, find locators, identify elements | Visual evidence, reporting, layout debugging |
| **When to use** | ⭐ Always use before interacting | Only on failure or an important milestone |
| **Has ref to interact** | ✅ Yes — use ref to click/type/hover | ❌ No — it is just an image |
| **Speed** | Fast | Slower |

**Rule:** Prefer `snapshot` for analysis, use `screenshot` for evidence.

---

## UI Inspection Process

### 1. Open & Prepare the page

```
browser_navigate → target URL
browser_resize → 1920 × 1080
browser_wait_for → wait for a page-loaded indicator (text or time)
```

If the page requires login:
- Ask the user for credentials OR use an existing fixture in the project
- **Do NOT read the `.env` file directly** (security rule)

### 2. Collect the DOM Structure

```
browser_snapshot → accessibility tree
```

From the snapshot, identify:
- **Main elements:** buttons, inputs, links, headings, tables
- **Important attributes:** role, name, label, placeholder, testid
- **Hierarchy:** parent → child relationships
- **State:** visible, enabled, disabled, checked, expanded

### 3. Identify Locators

For each element that needs a locator, apply the **priority order** per framework:

**Playwright:**

| Priority | Locator | Example | When to use |
|---|---|---|---|
| 1 ⭐ | `getByRole()` | `getByRole('button', {name: 'Submit'})` | Element with a clear role + accessible name |
| 2 | `getByLabel()` | `getByLabel('Email')` | Form input with a label |
| 3 | `getByPlaceholder()` | `getByPlaceholder('Enter email')` | Input with a placeholder, no label |
| 4 | `getByText()` | `getByText('Welcome back')` | Unique text content |
| 5 | `getByTestId()` | `getByTestId('submit-btn')` | Element with a data-testid attribute |
| 6 | CSS | `page.locator('.submit-button')` | No suitable semantic option |
| 7 | XPath | `page.locator('//div[@class="x"]')` | Last resort — avoid using |

**Selenium:**

| Priority | Locator | Example |
|---|---|---|
| 1 ⭐ | `By.id()` | `By.id("email")` |
| 2 | `By.cssSelector("[data-testid]")` | `By.cssSelector("[data-testid='submit']")` |
| 3 | `By.name()` | `By.name("username")` |
| 4 | `By.cssSelector()` | `By.cssSelector(".login-form button")` |
| 5 | `By.xpath()` | `By.xpath("//button[text()='Login']")` |

**Appium (Mobile):**

| Priority | Locator | Example |
|---|---|---|
| 1 ⭐ | Accessibility ID | `MobileBy.accessibilityId("loginButton")` |
| 2 | ID (resource-id) | `MobileBy.id("com.app:id/login_btn")` |
| 3 | Name | `MobileBy.name("Login")` |
| 4 | XPath (relative) | `MobileBy.xpath("//android.widget.Button[@text='Login']")` |

### 4. Verify the Locator

After identifying a locator, **you must verify** it against the real DOM:

```
browser_snapshot → find the element by ref
browser_click/type(ref) → try interacting
browser_snapshot → confirm the result
```

**A locator is accepted when:**
- [ ] It is unique on the page (matches only 1 element)
- [ ] It is stable across multiple reloads
- [ ] It does NOT contain dynamic classes (css-xxx, sc-xxx, MuiXxx-root)
- [ ] It does NOT contain positional xpath (//div[3]/button[2])
- [ ] It does NOT depend on auto-generated attributes

---

## Handling Special Situations

### Page requiring login
- Use an existing login fixture in the project or ask the user for credentials
- **Do NOT read .env directly**
- After logging in, navigate to the page you need to inspect

### Modal / Dialog / Popup
- A modal is usually an overlay on top of the main page
- `browser_snapshot` will show the modal content in the accessibility tree
- Interact with modal elements using the ref from the snapshot
- Wait for the modal animation to finish before interacting

### Iframe
- `browser_snapshot` may not show content inside an iframe
- Use `browser_evaluate` to switch into the iframe:
  ```javascript
  () => document.querySelector('iframe').contentDocument.body.innerHTML
  ```
- Or use the Playwright frame locator: `page.frameLocator('#iframe-id')`

### Shadow DOM
- Playwright `locator()` pierces the shadow DOM automatically
- Selenium needs `shadowRoot.findElement()`
- `browser_snapshot` may show shadow DOM content depending on the MCP version

### Dynamic Content (SPA / AJAX)
- Wait for content to load with `browser_wait_for(text)` before snapshotting
- If content loads lazily → scroll down first, then snapshot
- If content changes over time → take multiple snapshots

### Tables / Lists (many repeated elements)
- Identify a locator pattern for the row/cell
- Use `nth()` or `filter()` to target a specific element
- Playwright example: `page.getByRole('row').filter({hasText: 'John'}).getByRole('button', {name: 'Edit'})`

### Obscured element (Overlay / Toast)
- Check z-index, opacity, visibility in the DOM
- Wait for the overlay to disappear: `browser_wait_for(textGone: 'Loading...')`
- If a toast notification covers a button → wait for the toast timeout

---

## Anti-Patterns (FORBIDDEN)

| ❌ Wrong | ✅ Right | Reason |
|---|---|---|
| Guessing the locator from the feature name | Inspect the real DOM first, then take the locator | 100% accurate locator |
| Using a screenshot to choose a locator | Use the snapshot (accessibility tree) | The snapshot has a ref, the screenshot does not |
| Copying a locator from old code without verifying | Always verify the locator on the current browser | The DOM may have changed |
| Using a dynamic class `.css-1abc` | Use role/label/testid | Dynamic classes change every build |
| Using positional xpath `//div[3]` | Use relative xpath or CSS | Positional xpath breaks easily |
| Taking screenshots continuously | Only screenshot on failure or a milestone | Wastes resources, slow |
| Re-navigating when already on the correct page | Only navigate when the URL needs to change | Avoid unnecessary reloads |

---

## Output

This skill can return:

- **Locator recommendations** — a table of primary + fallback for each element
- **DOM analysis** — element structure, attributes, state, hierarchy
- **Page Object suggestions** — a class structure suited to the inspected page
- **Screenshots** — visual evidence at milestones
- **Debug findings** — the cause of element not found / click failure + how to fix

---

## Rules References

The agent MUST follow the detailed rules:

- `.agents/rules/locator_strategy.md` — Master locator priority map
- `.agents/rules/playwright_rules.md` — Playwright browser setup and locator rules
- `.agents/rules/selenium_rules.md` — Selenium locator and wait rules
- `.agents/rules/appium_rules.md` — Appium mobile locator rules
- `.agents/rules/automation_rules.md` — General automation best practices