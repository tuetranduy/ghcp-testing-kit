---
name: generate-automation-framework
description: Design and scaffold a complete automation framework. Supports Playwright, Selenium, Appium — Web, Mobile, API.
---

# Workflow: Design an Automation Framework

> **MANDATORY SKILL:** You MUST load and carefully read the skill **`framework-architect`** (at `.agents/skills/framework-architect/SKILL.md`) before starting. Also refer to the skill **`qa-automation-engineer`** to understand the shared automation rules.

This workflow helps the agent design, scaffold, and deliver a complete automation framework from scratch, tailored to the specific needs of the project.

## ⚠️ Execution principles

- **All output in English**
- **Do NOT guess** the tech stack — ask the user to confirm before scaffolding
- **You MUST create the `task.md` artifact** to track progress
- Every generated file must **compile/run immediately** — do not leave `// TODO` placeholders
- The framework must follow the design principles in the `framework-architect` skill

## Supported stacks

| Platform | Stack | Language | Runner | Report |
|---|---|---|---|---|
| 🌐 Web | Playwright | TypeScript | Playwright Test | HTML Report, Allure |
| 🌐 Web | Playwright | Java | TestNG / JUnit5 | Allure |
| 🌐 Web | Playwright | Python | Pytest | pytest-html, Allure |
| 🌐 Web | Selenium | Java | TestNG | Allure, ExtentReports |
| 🌐 Web | Selenium | Python | Pytest | pytest-html, Allure |
| 📱 Mobile | Appium | Java | TestNG | Allure, ExtentReports |
| 📱 Mobile | Appium | Python | Pytest | Allure |
| 🔌 API | REST Assured | Java | TestNG | Allure |
| 🔌 API | Playwright API | TypeScript | Playwright Test | HTML Report |

## Steps

### Step 1: Requirements Gathering (⏸️ CHECKPOINT)

1. **Ask the user** for the necessary information:

   | Question | Purpose | Default if unanswered |
   |---|---|---|
   | What application needs testing? (Web / Mobile / API / Hybrid) | Choose platform | Web |
   | Which framework? (Playwright / Selenium / Appium) | Choose tool | Playwright |
   | Which language? (TypeScript / Java / Python) | Choose language | TypeScript (Playwright), Java (Selenium/Appium) |
   | Project name? | Name the folder | `automation-framework` |
   | Do you need a CI/CD pipeline? | Generate pipeline config | Yes (GitHub Actions) |
   | Reporting tool? | Integrate reporting | Stack default |
   | Test the API in parallel? | Add API testing layer | No |
   | Parallel execution? | Configure parallelism | No |

2. **Confirm again** with the user before scaffolding:
   ```
   📋 Summary of the framework to create:
   - Platform: Web
   - Framework: Playwright
   - Language: TypeScript
   - Runner: Playwright Test
   - Report: HTML Report + Allure
   - CI/CD: GitHub Actions
   - Project name: my-automation
   
   Do you confirm so I can start scaffolding?
   ```

3. **Wait for user confirmation** before moving to Step 2

### Step 2: Scaffold Project Structure (Foundation)

1. **Create the `task.md` artifact** to track the checklist:
   ```markdown
   # Framework Setup Progress
   - [x] Step 1: Requirements gathering
   - [ ] Step 2: Scaffold project structure
   - [ ] Step 3: Generate base classes
   - [ ] Step 4: Generate example tests
   - [ ] Step 5: Configure reporting & CI/CD
   - [ ] Step 6: Verify & Deliver
   ```

2. **Create the project folders** following the template in the `framework-architect` skill:
   - Refer to the **Project Structure Templates** section in SKILL.md
   - Create all folders + root configuration files

3. **Generate build configuration files** (per stack):

   **Playwright + TypeScript:**
   - `package.json` — dependencies: `@playwright/test`, appropriate devDependencies
   - `playwright.config.ts` — baseURL, viewport (1920x1080), timeout, retries, reporter
   - `tsconfig.json` — paths, strict mode
   - `.env.example` — template environment variables

   **Selenium + Java:**
   - `pom.xml` — dependencies: selenium-java, testng, webdrivermanager, allure-testng, log4j
   - `testng.xml` — suite configuration, listeners
   - `log4j2.xml` — logging configuration

   **Appium + Java:**
   - `pom.xml` — dependencies: appium-java-client, selenium-java, testng, allure
   - `testng.xml` — suite configuration
   - Capabilities config file (JSON/YAML) for Android + iOS

   **Playwright + Python:**
   - `requirements.txt` — playwright, pytest, pytest-playwright, allure-pytest
   - `pyproject.toml` — pytest config, tool settings
   - `conftest.py` — root fixtures, browser setup

4. **Create a suitable .gitignore** (node_modules, target, __pycache__, .env, reports...)
5. **Create README.md** with instructions:
   - Prerequisites (Node.js, Java, Python version)
   - Installation steps
   - How to run tests
   - Project structure overview
   - Conventions (naming, coding standards)

### Step 3: Generate Core Classes (Base Layer)

1. **Configuration Management:**

   | Stack | File | Content |
   |---|---|---|
   | Playwright TS | `src/utils/env.config.ts` | Read `.env`, export typed config object |
   | Selenium Java | `src/main/.../config/ConfigReader.java` | Read properties file, singleton pattern |
   | Appium Java | `src/main/.../config/AppConfig.java` | Read capabilities JSON, env variables |
   | Playwright Python | `src/utils/config.py` | Read `.env` with `python-dotenv` |

2. **Browser / Driver Management:**

   | Stack | File | Content |
   |---|---|---|
   | Playwright TS | `playwright.config.ts` + fixtures | Browser config in config, auth in fixtures |
   | Selenium Java | `DriverFactory.java` | Factory pattern, WebDriverManager, thread-safe |
   | Appium Java | `AppiumDriverFactory.java` + `CapabilitiesManager.java` | Appium server URL, desired capabilities per device |
   | Playwright Python | `conftest.py` | Fixtures for browser, context, page |

3. **Base Page / Screen class:**
   - Common methods: `navigate()`, `click()`, `type()`, `getText()`, `isVisible()`
   - Built-in smart waits (NO hard sleep)
   - Screenshot on failure
   - Logging for each action

4. **Base Test class:**
   - Setup: initialize browser/driver, navigate to baseURL
   - Teardown: close browser/driver, capture screenshot on failure
   - Test lifecycle hooks (beforeAll, afterAll, beforeEach, afterEach)

5. **Utilities:**
   - `TestDataGenerator` — generate unique + traceable email, username, phone
   - `WaitHelper` — custom wait conditions (if needed beyond built-in)
   - `ScreenshotUtil` — capture + attach to report
   - `Logger` — structured logging (Log4j / winston / Python logging)

### Step 4: Generate Example Tests (Validation Layer)

1. **Create at least 1 example Page Object:**
   - `LoginPage` (or `LoginScreen` for mobile) with real locators + methods
   - Locators use placeholders that clearly indicate they must be replaced:
     ```typescript
     // REPLACE: Update locator after inspecting actual DOM
     readonly usernameInput = this.page.getByLabel('Username');
     readonly passwordInput = this.page.getByLabel('Password');
     readonly loginButton = this.page.getByRole('button', { name: 'Login' });
     ```

2. **Create at least 1 example Test:**
   - `LoginTest` — happy path demo (successful login)
   - Fully structured: Arrange → Act → Assert
   - Assertion with a clear message
   - Test data using TestDataGenerator (if applicable)

3. **Create 1 example data-driven test** (if appropriate):
   - Read data from a JSON/YAML/CSV file
   - Parameterized test with multiple data sets

### Step 5: Configure Reporting & CI/CD (Integration Layer)

1. **Reporting setup:**

   | Stack | Report | Configuration |
   |---|---|---|
   | Playwright TS | HTML + Allure | `reporter` in playwright.config.ts |
   | Selenium Java | Allure | `allure-testng` dependency + `@Step`, `@Attachment` annotations |
   | Appium Java | Allure + ExtentReports | `allure-testng` + `ExtentManager` singleton |
   | Playwright Python | pytest-html + Allure | `conftest.py` hooks + pytest addopts |

   - Screenshot auto-attach on failure
   - Test step logging in the report

2. **CI/CD Pipeline** (if requested by the user):

   **GitHub Actions template:**
   ```yaml
   # Generate the file .github/workflows/<framework>.yml
   # Content depends on the stack: install deps → run tests → upload report
   ```

   - Install dependencies
   - Run tests (headless mode for CI)
   - Upload test report as artifact
   - Parallel execution (if requested)
   - Environment variables from GitHub Secrets

3. **Docker support** (optional — only if requested by the user):
   - Dockerfile for the test execution environment
   - docker-compose.yml if a Selenium Grid / Appium server is needed

### Step 6: Verify & Deliver (Quality Gate)

1. **Check that the framework builds:**
   ```bash
   # Playwright TS
   npm install && npx playwright install && npx playwright test --list

   # Selenium Java
   mvn clean compile

   # Appium Java
   mvn clean compile

   # Playwright Python
   pip install -r requirements.txt && playwright install
   ```

2. **Run the example test** to verify the framework works:
   - If PASS → the framework is ready
   - If FAIL due to a missing app/URL → acceptable (note it in the README)
   - If FAIL due to a framework code error → fix it immediately

3. **Review checklist** before delivery:
   - [ ] Project structure matches the template
   - [ ] All dependencies declared
   - [ ] Config management works (reads .env)
   - [ ] Base Page/Test has all common methods
   - [ ] POM pattern is followed
   - [ ] Smart waits — no hard sleep
   - [ ] Example test runs (or runs once an app is available)
   - [ ] README.md has complete instructions
   - [ ] .gitignore covers correctly
   - [ ] Reporting integrated
   - [ ] No debug logs, commented code, or TODO placeholders

4. **Update `task.md`** with the completion status

## Handling special situations

| Situation | How to handle |
|---|---|
| **User wants hybrid (Web + Mobile)** | Create a shared layer (utils, config) + separate pages vs screens |
| **User wants hybrid (Web + API)** | Add a parallel API layer (api helpers + api tests folder) |
| **User has a legacy framework to refactor** | Read the current code → propose a migration plan → refactor incrementally |
| **User wants multi-browser** | Configure parallel projects in playwright.config.ts or the test suite XML |
| **User wants multi-device (Appium)** | CapabilitiesManager supports loading from JSON per device (Android/iOS) |
| **User does not know which stack to choose** | Suggest based on: team skills, project type, CI infra |

## Output

- **Project structure** complete (all folders + files)
- **Build config** (package.json / pom.xml / requirements.txt)
- **Framework config** (playwright.config.ts / testng.xml / conftest.py)
- **Base classes** (BasePage, BaseTest, DriverFactory, ConfigReader)
- **Utilities** (TestDataGenerator, WaitHelper, ScreenshotUtil, Logger)
- **Example Page Object + Test** (LoginPage + LoginTest)
- **Reporting integration** (Allure / HTML Report)
- **CI/CD pipeline** (GitHub Actions template)
- **README.md** (setup guide + project overview)
- **Artifact `task.md`** (progress checklist)