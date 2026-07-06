# Master Framework for E2E Automation (Generalized)

**Workflow:** generate-automation-from-testcases or generate-automation-from-ui-flow
**Skill:** qa-automation-engineer + framework-architect

---

## Goal

Build an Automation system that is scalable, maintainable, and reports professionally. Instead of "writing standalone tests", we build a solid **base Framework** from the start.

## How to use

Provide the appropriate architecture to the AI **in Step 0/1** so it knows where the source files live.
- **New project:** The AI uses the template below to scaffold.
- **Existing project:** Describe the current structure and the AI will follow it.

---

## Supported Stacks

### 🌐 Web Automation

| Stack                       | Language   | Runner          | Report                | Build Tool     |
| --------------------------- | ---------- | --------------- | --------------------- | -------------- |
| **Playwright + TypeScript** | TypeScript | Playwright Test | HTML Report, Allure   | npm            |
| **Playwright + Python**     | Python     | Pytest          | Allure, pytest-html   | pip            |
| **Selenium + Java**         | Java       | TestNG          | Allure, ExtentReports | Maven / Gradle |
| **Selenium + Python**       | Python     | Pytest          | Allure, pytest-html   | pip            |

### 📱 Mobile Automation

| Stack               | Language | Runner | Report                | Build Tool     |
| ------------------- | -------- | ------ | --------------------- | -------------- |
| **Appium + Java**   | Java     | TestNG | Allure, ExtentReports | Maven / Gradle |
| **Appium + Python** | Python   | Pytest | Allure, pytest-html   | pip            |

### 🔌 API Automation

| Stack                 | Language   | Runner          | Report      |
| --------------------- | ---------- | --------------- | ----------- |
| **REST Assured**      | Java       | TestNG          | Allure      |
| **Playwright API**    | TypeScript | Playwright Test | HTML Report |
| **Requests + Pytest** | Python     | Pytest          | Allure      |

---

## Standard Architecture

### 1. Playwright + TypeScript

```text
project-root/
├── playwright.config.ts          # Playwright configuration
├── package.json                  # Dependencies + npm scripts
├── .env.example                  # Environment template (NO real credentials)
├── .gitignore
├── README.md
├── src/
│   ├── pages/                    # Page Object classes
│   │   ├── base.page.ts          # Base page — common methods (wait, click, type, screenshot)
│   │   ├── login.page.ts
│   │   └── dashboard.page.ts
│   ├── fixtures/                 # Custom fixtures
│   │   ├── auth.fixture.ts       # Authentication fixture
│   │   └── base.fixture.ts       # Extended test with all fixtures
│   ├── utils/                    # Helpers & utilities
│   │   ├── test-data.ts          # Data generators (unique + traceable)
│   │   ├── env.config.ts         # Environment config reader
│   │   └── helpers.ts            # Common helper functions
│   └── tests/                    # Test specs (grouped by feature)
│       ├── auth/
│       │   └── login.spec.ts
│       └── dashboard/
│           └── dashboard.spec.ts
├── test-data/                    # External test data (JSON/YAML)
│   └── users.json
└── .github/
    └── workflows/
        └── playwright.yml        # CI pipeline template
```

### 2. Playwright + Python (Pytest)

```text
project-root/
├── pyproject.toml                # Python project config
├── requirements.txt              # Dependencies
├── conftest.py                   # Root fixtures + browser setup
├── .env.example
├── .gitignore
├── README.md
├── src/
│   ├── pages/
│   │   ├── base_page.py          # Base page — common methods
│   │   ├── login_page.py
│   │   └── dashboard_page.py
│   ├── utils/
│   │   ├── config.py             # Env config reader
│   │   ├── test_data.py          # Data generators
│   │   └── helpers.py
│   └── tests/
│       ├── conftest.py           # Test-level fixtures
│       ├── test_login.py
│       └── test_dashboard.py
├── test-data/
│   └── users.json
└── .github/
    └── workflows/
        └── playwright.yml
```

### 3. Selenium + Java (Maven + TestNG)

```text
project-root/
├── pom.xml                       # Maven config + dependencies
├── testng.xml                    # TestNG suite config
├── .env.example
├── .gitignore
├── README.md
├── src/
│   ├── main/java/
│   │   └── com/project/
│   │       ├── pages/            # Page Object classes
│   │       │   ├── BasePage.java         # Base page — common methods
│   │       │   ├── LoginPage.java
│   │       │   └── DashboardPage.java
│   │       ├── drivers/          # Driver management
│   │       │   └── DriverFactory.java
│   │       ├── config/           # Configuration
│   │       │   └── ConfigReader.java
│   │       └── utils/            # Utilities
│   │           ├── WaitHelper.java
│   │           ├── ScreenshotUtil.java
│   │           └── TestDataGenerator.java
│   └── test/java/
│       └── com/project/
│           ├── base/
│           │   └── BaseTest.java         # Test lifecycle hooks
│           └── tests/
│               ├── LoginTest.java
│               └── DashboardTest.java
├── test-data/
│   └── users.json
└── .github/
    └── workflows/
        └── selenium.yml
```

### 4. Selenium + Python (Pytest)

```text
project-root/
├── pyproject.toml
├── requirements.txt
├── conftest.py                   # Root fixtures + driver setup
├── .env.example
├── .gitignore
├── README.md
├── src/
│   ├── pages/
│   │   ├── base_page.py
│   │   ├── login_page.py
│   │   └── dashboard_page.py
│   ├── drivers/
│   │   └── driver_factory.py     # WebDriver factory
│   ├── utils/
│   │   ├── config.py
│   │   ├── test_data.py
│   │   ├── wait_helper.py
│   │   └── screenshot_util.py
│   └── tests/
│       ├── conftest.py
│       ├── test_login.py
│       └── test_dashboard.py
├── test-data/
│   └── users.json
└── .github/
    └── workflows/
        └── selenium.yml
```

### 5. Appium + Java (Maven + TestNG)

```text
project-root/
├── pom.xml
├── testng.xml
├── .env.example
├── .gitignore
├── README.md
├── src/
│   ├── main/java/
│   │   └── com/project/
│       │   ├── screens/          # Screen Objects (equivalent to Page Objects)
│   │       │   ├── BaseScreen.java       # Base screen — common mobile methods

│   │       │   ├── LoginScreen.java
│   │       │   └── HomeScreen.java
│   │       ├── drivers/          # Appium driver management
│   │       │   ├── AppiumDriverFactory.java
│   │       │   └── CapabilitiesManager.java
│   │       ├── config/
│   │       │   └── AppConfig.java
│   │       └── utils/
│   │           ├── MobileGestures.java   # Swipe, scroll, tap helpers
│   │           ├── ScreenshotUtil.java
│   │           └── TestDataGenerator.java
│   └── test/java/
│       └── com/project/
│           ├── base/
│           │   └── BaseTest.java
│           └── tests/
│               ├── LoginTest.java
│               └── HomeTest.java
├── apps/                         # APK/IPA files
│   └── .gitkeep
├── test-data/
│   └── users.json
└── .github/
    └── workflows/
        └── appium.yml
```

### 6. Appium + Python (Pytest)

```text
project-root/
├── pyproject.toml
├── requirements.txt
├── conftest.py                   # Root fixtures + Appium driver setup
├── .env.example
├── .gitignore
├── README.md
├── src/
│   ├── screens/
│   │   ├── base_screen.py
│   │   ├── login_screen.py
│   │   └── home_screen.py
│   ├── drivers/
│   │   ├── appium_driver_factory.py
│   │   └── capabilities_manager.py
│   ├── utils/
│   │   ├── config.py
│   │   ├── test_data.py
│   │   ├── mobile_gestures.py
│   │   └── screenshot_util.py
│   └── tests/
│       ├── conftest.py
│       ├── test_login.py
│       └── test_home.py
├── apps/
│   └── .gitkeep
├── test-data/
│   └── users.json
└── .github/
    └── workflows/
        └── appium.yml
```

### 7. REST Assured (Java + TestNG)

```text
project-root/
├── pom.xml
├── testng.xml
├── .env.example
├── .gitignore
├── README.md
├── src/
│   ├── main/java/
│   │   └── com/project/
│   │       ├── api/              # API client classes
│   │       │   ├── BaseApi.java          # Base API — common request methods
│   │       │   ├── AuthApi.java
│   │       │   └── UserApi.java
│   │       ├── models/           # POJO / DTO classes
│   │       │   ├── UserRequest.java
│   │       │   └── UserResponse.java
│   │       ├── config/
│   │       │   └── ApiConfig.java
│   │       └── utils/
│   │           ├── TestDataGenerator.java
│   │           └── JsonHelper.java
│   └── test/java/
│       └── com/project/
│           ├── base/
│           │   └── BaseApiTest.java
│           └── tests/
│               ├── AuthApiTest.java
│               └── UserApiTest.java
├── test-data/
│   ├── payloads/                 # Request body templates (JSON)
│   │   └── create_user.json
│   └── schemas/                  # JSON Schema validation
│       └── user_response_schema.json
└── .github/
    └── workflows/
        └── api-tests.yml
```

---

## Component Checklist (Mandatory)

Every framework MUST include the following components:

| #   | Component                       | Required      | Description                                                    |
| --- | ------------------------------- | ------------- | -------------------------------------------------------------- |
| 1   | **Project Structure**           | ✅             | Clear directories, separating pages/tests/utils/config         |
| 2   | **Config Management**           | ✅             | Environment via `.env` + config file — NO hardcoding           |
| 3   | **Browser / Driver Management** | ✅             | Factory pattern for browser/driver setup                       |
| 4   | **Base Classes**                | ✅             | BasePage/BaseScreen containing common methods                  |
| 5   | **Page Object Model**           | ✅             | Each page → 1 class, locators declared at the top of the class |
| 6   | **Test Data Management**        | ✅             | Data factory + external data (JSON/YAML) + unique/traceable    |
| 7   | **Utilities**                   | ✅             | Wait helpers, screenshot utils, logger, string generators      |
| 8   | **Reporting**                   | ✅             | At least 1 report tool + screenshot on failure                 |
| 9   | **CI/CD Pipeline**              | 🟡 Recommended | GitHub Actions / GitLab CI / Jenkins template                  |

---

## Design Principles

1. **DRY** — Write each piece of logic once, reuse it via Base classes and Utils
2. **Single Responsibility** — Pages hold only UI interaction, Tests hold only test logic
3. **Open/Closed** — Easy to extend (add a page, add a test) without touching the core
4. **Configuration over Code** — Env, browser, and timeout are managed via config, not hardcoded
5. **Fail Fast, Log Rich** — Screenshot on failure, structured logging, clear assertion messages

---

## Anti-Patterns (FORBIDDEN)

| ❌ Anti-Pattern                                | ✅ Correct approach                         |
| --------------------------------------------- | ------------------------------------------ |
| Hardcode URL/credentials in code              | Read from `.env` or a config file          |
| Locator inline in a test                      | Declare it in the Page class               |
| `Thread.sleep()` / `waitForTimeout()`         | Smart waits (`expect()`, `WebDriverWait`)  |
| Global mutable state                          | Isolated fixtures/setup per test           |
| Monolithic test file (1 file with 500+ lines) | Split by module/feature                    |
| `System.out.println()` / `console.log()`      | Logger framework (Log4j, winston, logging) |
| Guessing locators without inspecting the DOM  | Open the browser and inspect the real DOM  |

---

## References

- **Detailed skill:** `.agents/skills/framework-architect/SKILL.md`
- **Rules:** `.agents/rules/automation_rules.md`, `.agents/rules/locator_strategy.md`
- **Workflow scaffold:** generate-automation-framework
