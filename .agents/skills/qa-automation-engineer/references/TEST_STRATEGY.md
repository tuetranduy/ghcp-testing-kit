# Test Strategy

## Usage Guide

This file defines the testing strategy for the project. The agent refers to this file to understand the scope, priorities, and approach when generating test cases or automation scripts.

> ⚠️ **You need to update this file** for each specific project. Below is a template.

---

## Testing Objectives

- Ensure software quality across the different testing levels
- Detect defects early in the development lifecycle
- Maintain a stable regression suite for CI/CD

## Scope of Testing

| Test Type             | Applicable | Tool/Framework         |
| --------------------- | ---------- | ---------------------- |
| UI Functional Testing | ✅          | Selenium / Playwright  |
| API Testing           | ✅          | REST Assured / Postman |
| Unit Testing          | ✅          | JUnit / TestNG         |
| Integration Testing   | ✅          | TestNG + REST Assured  |
| Performance Testing   | ⬜          | JMeter / k6            |
| Security Testing      | ⬜          | OWASP ZAP              |
| Mobile Testing        | ⬜          | Appium                 |

## Test Automation Strategy

### Framework Architecture
- **Design Pattern:** Page Object Model (POM)
- **Language:** Java
- **Test Runner:** TestNG
- **Build Tool:** Maven
- **Reporting:** Allure / ExtentReports

### Automation Scope
- Smoke tests: Cover the happy path of the main features
- Regression tests: Cover all test cases that have passed
- Data-driven tests: Use external data sources (Excel, CSV, JSON)

## Test Data Management

- Use random data with prefix + timestamp for traceability
- Separate test data from test logic
- Do not hard-code credentials in the code

## Execution Plan

| Phase       | Description     | Trigger        |
| ----------- | --------------- | -------------- |
| Smoke Test  | Main happy path | Every build    |
| Regression  | Full suite      | Before release |
| Integration | API + UI        | Daily          |

## Test Environment

- Tests run on the Staging environment
- The CI/CD pipeline runs in headless mode
- Local debugging runs in headed mode (viewport 1920x1080)
