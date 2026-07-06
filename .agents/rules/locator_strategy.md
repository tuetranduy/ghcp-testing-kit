# Locator Selection Strategy (Applies to Every Framework)

> The stability and readability of a locator determine the health of an automation framework.
> Core principle: NEVER select an element based on a DOM structure tied to styling. Build locators on semantic attributes.

## 1. Master Priority Map

Priority order from highest to lowest:

1. Accessibility / Aria attributes (semantic, screen-reader friendly)
2. Dedicated test attributes (`data-testid`, `data-test`, `data-qa`)
3. Primary identifier attributes (`id`, `resource-id`, `name`)
4. Framework-specific semantic functions (Playwright: `getByRole`, `getByLabel`...)
5. CSS Selector
6. XPath (last resort)

## 2. Stability Rules

Every locator must ensure:
- It matches **exactly 1 element** on the page (unique in scope).
- It survives UI changes — unaffected when the DOM changes layout (adding/removing wrapper divs, changing flexbox).

**FORBIDDEN to use:**
- Dynamic / temporary hashed CSS class names (e.g., `css-1n2xyz-btn`)
- `nth-child`, `nth-of-type` strings when a better option exists
- Framework auto-generated IDs
- Absolute position-based XPath (e.g., `//div[3]/div[2]/form/button`)

## 3. Locator Verification Process

Before putting a locator into the code, you must check:

1. Does the locator match **exactly 1 element** in the DOM?
2. Is the matched element one the user can interact with? (not a shadow DOM overlay)
3. After reloading / re-navigating the page, is the locator still correct?
4. Try it across multiple page states (loading, loaded, with data, without data) — is the locator stable?

## 4. Locators by Framework

For framework-specific locator details, see:
- Playwright: `.agents/rules/playwright_rules.md` (Section 3)
- Selenium: `.agents/rules/selenium_rules.md` (Section 1)
- Appium: `.agents/rules/appium_rules.md` (Section 1)
