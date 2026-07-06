---
name: framework-architect
description: A skill to design and scaffold a complete automation framework for Playwright, Selenium, and Appium — including project structure, base classes, config management, reporting, and CI/CD integration.
---

# Framework Architect

## Description

A specialized skill that helps the agent design, scaffold, and set up an automation framework from scratch. It supports multiple platforms (Web, Mobile, API) with the most popular frameworks.

The agent can:

- Design a project structure following best practices
- Generate base classes, config management, driver/browser management
- Integrate reporting (Allure, HTML Report, Playwright Report)
- Configure a CI/CD pipeline (GitHub Actions, GitLab CI, Jenkins)
- Generate Page Object Model templates, fixtures, helpers
- Create configuration files (package.json, pom.xml, build.gradle, playwright.config.ts)

---

## When to Use

Use this skill when:

- The user asks to create/design a new automation framework
- The user needs to scaffold a project structure for test automation
- The user wants to standardize an existing framework
- The user needs to integrate reporting or CI/CD into a framework
- The user asks about best practices for framework design

Trigger keywords: "create framework", "design framework", "scaffold project", "new project"

---

## Supported Stacks

### 🌐 Web Automation

| Stack | Language | Runner | Report | Build Tool |
|---|---|---|---|---|
| **Playwright + TypeScript** | TypeScript | Playwright Test | HTML Report, Allure | npm |
| **Playwright + Java** | Java | TestNG / JUnit5 | Allure Report | Maven / Gradle |
| **Playwright + Python** | Python | Pytest | Allure, pytest-html | pip |
| **Selenium + Java** | Java | TestNG | Allure, ExtentReports | Maven / Gradle |
| **Selenium + Python** | Python | Pytest | Allure, pytest-html | pip |

### 📱 Mobile Automation

| Stack | Language | Runner | Report | Build Tool |
|---|---|---|---|---|
| **Appium + Java** | Java | TestNG | Allure, ExtentReports | Maven / Gradle |
| **Appium + Python** | Python | Pytest | Allure, pytest-html | pip |

### 🔌 API Automation

| Stack | Language | Runner | Report |
|---|---|---|---|
| **REST Assured** | Java | TestNG | Allure |
| **Playwright API** | TypeScript | Playwright Test | HTML Report |
| **Requests + Pytest** | Python | Pytest | Allure |

---

## Framework Components

Every framework MUST include the following components (customized per stack):

### 1. Project Structure (Mandatory)
- A clear folder structure, separating pages/tests/utils/config
- A README.md file with setup + test run instructions
- An appropriate .gitignore file

### 2. Configuration Management (Mandatory)
- Manage environments (dev/staging/prod) through a config file or .env
- Centralized config — no hardcoded values in tests
- Sensitive data (credentials) through environment variables, NEVER committed to the repo

### 3. Browser / Driver Management (Mandatory)
- **Playwright:** playwright.config.ts / conftest.py with browser setup
- **Selenium:** WebDriverManager or Driver Factory pattern
- **Appium:** Desired Capabilities factory, Appium server config

### 4. Base Classes (Mandatory)
- Base Page — contains common methods (wait, click, type, screenshot)
- Base Test — contains setup/teardown, test lifecycle hooks
- No hardcoded waits — use smart waits only

### 5. Page Object Model (Mandatory)
- Each page/screen → 1 Page class
- Locators declared at the top of the class, not inline in the test
- Methods describe user behavior (not DOM operations)

### 6. Test Data Management (Mandatory)
- Data factory / builder pattern for test data
- External data (JSON/YAML/CSV) for data-driven tests
- Unique + traceable data (timestamp/random prefix)

### 7. Utilities (Mandatory)
- Wait helpers (smart waits, custom conditions)
- Screenshot utilities (capture on failure)
- Logger (structured logging, no print/console.log)
- Date/Time helpers, String generators

### 8. Reporting (Mandatory)
- Integrate at least 1 reporting tool
- Attach screenshot on failure
- Test execution summary (pass/fail/skip counts)

### 9. CI/CD Pipeline (Optional — but recommended)
- GitHub Actions / GitLab CI / Jenkins pipeline template
- Parallel execution config
- Artifact upload (reports, screenshots)

---

## Project Structure Templates

### Playwright + TypeScript

```
project-root/
├── playwright.config.ts        # Playwright configuration
├── package.json                # Dependencies + scripts
├── .env.example                # Environment template
├── .gitignore
├── README.md
├── src/
│   ├── pages/                  # Page Object classes
│   │   ├── base.page.ts        # Base page (common methods)
│   │   ├── login.page.ts
│   │   └── dashboard.page.ts
│   ├── fixtures/               # Custom fixtures
│   │   ├── auth.fixture.ts     # Authentication fixture
│   │   └── base.fixture.ts     # Extended test with all fixtures
│   ├── utils/                  # Helpers & utilities
│   │   ├── test-data.ts        # Data generators
│   │   ├── env.config.ts       # Environment config reader
│   │   └── helpers.ts          # Common helper functions
│   └── tests/                  # Test specs
│       ├── auth/
│       │   └── login.spec.ts
│       └── dashboard/
│           └── dashboard.spec.ts
├── test-data/                  # External test data (JSON/YAML)
│   └── users.json
└── .github/
    └── workflows/
        └── playwright.yml      # CI pipeline
```

### Selenium + Java (Maven + TestNG)

```
project-root/
├── pom.xml                     # Maven config + dependencies
├── testng.xml                  # TestNG suite config
├── .env.example
├── .gitignore
├── README.md
├── src/
│   ├── main/java/
│   │   └── com/project/
│   │       ├── pages/          # Page Object classes
│   │       │   ├── BasePage.java
│   │       │   ├── LoginPage.java
│   │       │   └── DashboardPage.java
│   │       ├── drivers/        # Driver management
│   │       │   └── DriverFactory.java
│   │       ├── config/         # Configuration
│   │       │   └── ConfigReader.java
│   │       └── utils/          # Utilities
│   │           ├── WaitHelper.java
│   │           ├── ScreenshotUtil.java
│   │           └── TestDataGenerator.java
│   └── test/java/
│       └── com/project/
│           ├── base/
│           │   └── BaseTest.java
│           └── tests/
│               ├── LoginTest.java
│               └── DashboardTest.java
├── test-data/
│   └── users.json
└── .github/
    └── workflows/
        └── selenium.yml
```

### Appium + Java (Maven + TestNG)

```
project-root/
├── pom.xml
├── testng.xml
├── .env.example
├── .gitignore
├── README.md
├── src/
│   ├── main/java/
│   │   └── com/project/
│   │       ├── screens/        # Screen Object classes (mobile POM)
│   │       │   ├── BaseScreen.java
│   │       │   ├── LoginScreen.java
│   │       │   └── HomeScreen.java
│   │       ├── drivers/        # Appium driver management
│   │       │   ├── AppiumDriverFactory.java
│   │       │   └── CapabilitiesManager.java
│   │       ├── config/
│   │       │   └── AppConfig.java
│   │       └── utils/
│   │           ├── MobileGestures.java    # Swipe, scroll, tap
│   │           ├── ScreenshotUtil.java
│   │           └── TestDataGenerator.java
│   └── test/java/
│       └── com/project/
│           ├── base/
│           │   └── BaseTest.java
│           └── tests/
│               ├── LoginTest.java
│               └── HomeTest.java
├── apps/                       # APK/IPA files
│   └── .gitkeep
├── test-data/
│   └── users.json
└── .github/
    └── workflows/
        └── appium.yml
```

### Playwright + Python (Pytest)

```
project-root/
├── playwright.config.py        # Pytest-playwright config (if any)
├── pyproject.toml              # Python project config
├── requirements.txt            # Dependencies
├── conftest.py                 # Root fixtures + browser setup
├── .env.example
├── .gitignore
├── README.md
├── src/
│   ├── pages/
│   │   ├── base_page.py
│   │   ├── login_page.py
│   │   └── dashboard_page.py
│   ├── utils/
│   │   ├── config.py           # Env config reader
│   │   ├── test_data.py        # Data generators
│   │   └── helpers.py
│   └── tests/
│       ├── conftest.py         # Test-level fixtures
│       ├── test_login.py
│       └── test_dashboard.py
├── test-data/
│   └── users.json
└── .github/
    └── workflows/
        └── playwright.yml
```

---

## Design Principles

1. **DRY (Don't Repeat Yourself)** — Each piece of logic is written once and reused through Base classes and Utils
2. **Single Responsibility** — Each class/module does one thing (a Page contains only UI interaction, a Test contains only test logic)
3. **Open/Closed** — The framework is easy to extend (add a page, add a test) without modifying the core
4. **Configuration over Code** — Env, browser, timeout... managed through config, not hardcoded
5. **Fail Fast, Log Rich** — Screenshot on failure, structured logging, clear assertion messages

---

## Anti-Patterns (FORBIDDEN)

| ❌ Anti-Pattern | ✅ Correct approach |
|---|---|
| Hardcoded URL/credentials in code | Read from .env or a config file |
| Locator inline in the test | Declare it in the Page class |
| `Thread.sleep()` / `waitForTimeout()` | Smart waits (`expect()`, `WebDriverWait`) |
| Global mutable state | Isolated fixtures/setup per test |
| Monolithic test file (1 file with 500+ lines) | Split by module/feature |
| `System.out.println()` / `console.log()` | Logger framework (Log4j, winston, logging) |

---

## Rules References

The agent MUST follow the detailed rules:

- `.agents/rules/automation_rules.md` — General automation best practices
- `.agents/rules/locator_strategy.md` — Locator selection priority
- `.agents/rules/playwright_rules.md` — Playwright-specific rules
- `.agents/rules/selenium_rules.md` — Selenium-specific rules
- `.agents/rules/appium_rules.md` — Appium mobile automation rules
