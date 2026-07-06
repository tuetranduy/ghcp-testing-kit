# Appium-Specific Rules (Mobile Automation)

> Apply when automating mobile applications with Java and Appium.

## 1. Locator Priority Order

Use native, platform-specific locator strategies (iOS / Android) instead of web equivalents:

1. `accessibility id` — Cross-platform, the most stable
2. `resource-id` (Android) — Native Android attribute
3. `id` — Generic ID
4. `iOS predicate string` (iOS) — Fast, iOS-specific
5. `iOS class chain` (iOS) — iOS structural query
6. `xpath` — Last resort (slowest)

Correct examples:
```java
// Accessibility id — Cross-platform, always preferred
driver.findElement(AppiumBy.accessibilityId("login_button"));

// Android — resource-id
driver.findElement(AppiumBy.id("com.application.xyz:id/login_button"));

// iOS — Predicate String (fast)
driver.findElement(AppiumBy.iOSNsPredicateString("label == 'Login'"));

// iOS — Class Chain
driver.findElement(AppiumBy.iOSClassChain(
    "**/XCUIElementTypeButton[`label == 'Login'`]"
));
```

## 2. FORBIDDEN

- Absolute position-based XPath — any small layout change causes a failure:
  ```java
  // FORBIDDEN:
  driver.findElement(By.xpath(
      "//android.widget.FrameLayout[1]/android.widget.LinearLayout[2]/android.widget.Button[1]"
  ));
  ```
- Querying an off-screen element without scrolling first
- Interacting with a disabled element without checking its state
- Hardcoding animation wait times

## 3. Wait Strategy

**FORBIDDEN:**
- `Thread.sleep()` — In all cases

**USE:**
- Explicit Waits with `WebDriverWait`:
  ```java
  WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(15));

  // Wait for the element to be visible
  wait.until(ExpectedConditions.visibilityOfElementLocated(
      AppiumBy.accessibilityId("welcome_text")
  ));

  // Wait for the element to be clickable
  wait.until(ExpectedConditions.elementToBeClickable(
      AppiumBy.accessibilityId("submit_button")
  ));
  ```

- Handle scrolling to an element:
  ```java
  // Android — UiScrollable
  driver.findElement(AppiumBy.androidUIAutomator(
      "new UiScrollable(new UiSelector().scrollable(true))" +
      ".scrollIntoView(new UiSelector().text(\"Submit\"))"
  ));
  ```

## 4. Test Structure (TestNG)

```java
public class LoginMobileTest extends BaseTest {

    @BeforeMethod
    public void setUp() {
        // Initialize driver, capabilities...
    }

    @Test(groups = {"mobile", "regression"})
    public void testLoginSuccess() {
        // Arrange
        LoginScreen loginScreen = new LoginScreen(driver);
        String email = DataGenerator.generateEmail("loginMobile");

        // Act
        loginScreen.login(email, "ValidPass@123");

        // Assert
        HomeScreen homeScreen = new HomeScreen(driver);
        Assert.assertTrue(homeScreen.isWelcomeDisplayed(),
            "The Home screen must be displayed after logging in");
    }
}
```

Notes:
- Mobile uses **Screen Objects** (equivalent to Page Objects) — with the `Screen` suffix
- Examples: `LoginScreen.java`, `HomeScreen.java`, `SettingsScreen.java`

## 5. Mobile Testing Specifics

- **Screen rotation:** Test both portrait and landscape if the app supports it:
  ```java
  driver.rotate(ScreenOrientation.LANDSCAPE);
  ```
- **Background/Foreground:** Test the app when sent to the background and brought back:
  ```java
  driver.runAppInBackground(Duration.ofSeconds(5));
  ```
- **Push Notification:** Verify notifications with the Appium notification listener
- **Permission Dialog:** Handle permission request dialogs (camera, location...):
  ```java
  // Android — Auto-grant permissions in capabilities
  capabilities.setCapability("autoGrantPermissions", true);
  ```
