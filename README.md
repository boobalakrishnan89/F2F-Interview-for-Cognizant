# 🎯 Automation Test Engineer — In-Depth Interview Q&A

> Covers: Java, Selenium WebDriver, API Testing, REST Assured, Test Automation Frameworks, SQL, Agile Methodology, and Behavioral/Scenario-based questions aligned to the job responsibilities.

---

## Table of Contents

1. [Java (Core & Advanced)](#1-java-core--advanced)
2. [Selenium WebDriver](#2-selenium-webdriver)
3. [API Testing Concepts](#3-api-testing-concepts)
4. [REST Assured](#4-rest-assured)
5. [Test Automation Framework Design](#5-test-automation-framework-design)
6. [SQL for Testers](#6-sql-for-testers)
7. [Agile Methodology](#7-agile-methodology)
8. [Testing Fundamentals](#8-testing-fundamentals)
9. [Defect Management & Reporting](#9-defect-management--reporting)
10. [CI/CD & DevOps for QA](#10-cicd--devops-for-qa)
11. [Scenario-Based / Problem-Solving Questions](#11-scenario-based--problem-solving-questions)
12. [Behavioral / Situational Questions](#12-behavioral--situational-questions)

---

## 1. Java (Core & Advanced)

### Q1. What are the four pillars of OOP and how do you use them in test automation?

**Answer:**

| Pillar | Definition | Usage in Automation |
|---|---|---|
| **Encapsulation** | Bundling data and methods together, restricting direct access | Page Object classes — fields are `private`, exposed via public methods |
| **Inheritance** | Child class inherits properties/methods from parent | `BasePage` → `LoginPage`, `BasePage` holds common methods like `click()`, `waitForElement()` |
| **Polymorphism** | Same method behaves differently based on object | Method overloading (multiple `login()` signatures), method overriding (custom `click()` in child page) |
| **Abstraction** | Hiding complex implementation, showing only essential features | Interface `WebDriver` — you use `driver.findElement()` without knowing how Chrome/Firefox implements it internally |

**Example in code:**
```java
// Encapsulation + Abstraction
public class BasePage {
    private WebDriver driver; // encapsulated

    public BasePage(WebDriver driver) {
        this.driver = driver;
        PageFactory.initElements(driver, this);
    }

    protected void click(WebElement element) { // abstraction
        waitForClickable(element);
        element.click();
    }
}

// Inheritance + Polymorphism
public class LoginPage extends BasePage {
    @FindBy(id = "username") private WebElement usernameField;
    @FindBy(id = "password") private WebElement passwordField;

    public LoginPage(WebDriver driver) { super(driver); }

    public void login(String user, String pass) { /* overloaded method */ }
    public void login(String user, String pass, boolean rememberMe) { /* overloaded */ }
}
```

---

### Q2. Explain `String`, `StringBuilder`, and `StringBuffer`. When would you use each in automation?

**Answer:**

| Feature | `String` | `StringBuilder` | `StringBuffer` |
|---|---|---|---|
| **Mutability** | Immutable | Mutable | Mutable |
| **Thread Safety** | Yes (immutable) | No | Yes (synchronized) |
| **Performance** | Slow for concatenation | Fast | Moderate |
| **Use Case** | Storing test data, assertions | Building dynamic URLs/XPaths in loops | Multi-threaded report generation |

```java
// Building dynamic XPath — use StringBuilder
StringBuilder xpath = new StringBuilder("//div[@class='product']");
xpath.append("[contains(text(),'").append(productName).append("')]");
driver.findElement(By.xpath(xpath.toString()));

// String is fine for simple comparisons
String expectedTitle = "Dashboard";
Assert.assertEquals(driver.getTitle(), expectedTitle);
```

---

### Q3. What is the difference between `==` and `.equals()` in Java?

**Answer:**
- `==` compares **reference** (memory address) — are these the same object?
- `.equals()` compares **content/value** — do these objects hold the same data?

```java
String s1 = new String("Selenium");
String s2 = new String("Selenium");

System.out.println(s1 == s2);       // false (different objects in heap)
System.out.println(s1.equals(s2));  // true  (same content)

// In automation — ALWAYS use .equals() for String comparisons
String actualTitle = driver.getTitle();
Assert.assertTrue(actualTitle.equals("Home Page"));
// Or better: Assert.assertEquals(actualTitle, "Home Page");
```

---

### Q4. Explain Java Collections you use frequently in automation testing.

**Answer:**

| Collection | When I Use It |
|---|---|
| `ArrayList` | Storing lists of test data, web elements, error messages |
| `HashMap` | Storing key-value test data (username→password), configuration properties |
| `LinkedHashMap` | When insertion order matters (e.g., form fields to fill in order) |
| `HashSet` | Removing duplicate values from data sets, unique validation |
| `Properties` | Reading `.properties` config files (URLs, credentials) |

```java
// Reading test data from HashMap
Map<String, String> testData = new HashMap<>();
testData.put("username", "admin");
testData.put("password", "secret123");

loginPage.login(testData.get("username"), testData.get("password"));

// Collecting all links on a page
List<WebElement> allLinks = driver.findElements(By.tagName("a"));
List<String> linkTexts = new ArrayList<>();
for (WebElement link : allLinks) {
    linkTexts.add(link.getText());
}

// Using Set to find duplicates
Set<String> uniqueTexts = new HashSet<>(linkTexts);
if (uniqueTexts.size() != linkTexts.size()) {
    System.out.println("Duplicate links found!");
}
```

---

### Q5. What are functional interfaces and lambda expressions? How do you use them?

**Answer:**

A **functional interface** has exactly one abstract method (e.g., `Runnable`, `Predicate`, `Function`). **Lambda expressions** provide a concise way to implement them.

```java
// Without lambda
WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
wait.until(new ExpectedCondition<Boolean>() {
    @Override
    public Boolean apply(WebDriver d) {
        return d.getTitle().contains("Dashboard");
    }
});

// With lambda — cleaner
wait.until(d -> d.getTitle().contains("Dashboard"));

// Using Predicate for filtering test data
List<String> testUrls = Arrays.asList("/login", "/dashboard", "/admin", "/login");
List<String> filtered = testUrls.stream()
    .filter(url -> url.startsWith("/admin"))
    .collect(Collectors.toList());
```

---

### Q6. Explain exception handling in Java. How do you handle exceptions in automation?

**Answer:**

```java
// Custom exception for automation
public class ElementNotFoundException extends RuntimeException {
    public ElementNotFoundException(String message) {
        super(message);
    }
}

// Handling in page object
public class BasePage {
    public void clickElement(WebElement element, String elementName) {
        try {
            WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
            wait.until(ExpectedConditions.elementToBeClickable(element));
            element.click();
        } catch (TimeoutException e) {
            throw new ElementNotFoundException(
                elementName + " was not clickable within 10 seconds");
        } catch (StaleElementReferenceException e) {
            // Re-find and retry
            driver.navigate().refresh();
            element.click();
        } catch (Exception e) {
            takeScreenshot(elementName + "_click_failure");
            throw e; // re-throw after capturing evidence
        }
    }
}
```

**Key exceptions in Selenium:**

| Exception | Cause | How to Handle |
|---|---|---|
| `NoSuchElementException` | Element not found | Verify locator, add explicit wait |
| `StaleElementReferenceException` | DOM refreshed, element reference is stale | Re-find the element |
| `TimeoutException` | Wait condition not met within time | Increase timeout or fix condition |
| `ElementNotInteractableException` | Element exists but can't be interacted with | Scroll into view, wait for visibility |
| `WebDriverException` | Browser/driver issue | Check driver version compatibility |

---

### Q7. What are the access modifiers in Java and how do you decide which to use?

**Answer:**

| Modifier | Class | Package | Subclass | World |
|---|---|---|---|---|
| `private` | ✅ | ❌ | ❌ | ❌ |
| `default` (no keyword) | ✅ | ✅ | ❌ | ❌ |
| `protected` | ✅ | ✅ | ✅ | ❌ |
| `public` | ✅ | ✅ | ✅ | ✅ |

**In automation frameworks:**
- `private` — WebElement locators in Page Objects, utility helper fields
- `protected` — `WebDriver` instance in `BasePage` (so child pages can access)
- `public` — Page action methods (`login()`, `search()`), test methods
- `default` — Rarely used; sometimes for package-level test utilities

---

### Q8. Explain `static` keyword usage in automation.

**Answer:**

```java
public class DriverManager {
    // ThreadLocal for parallel execution — each thread gets its own driver
    private static ThreadLocal<WebDriver> driver = new ThreadLocal<>();

    public static WebDriver getDriver() {
        return driver.get();
    }

    public static void setDriver(WebDriver webDriver) {
        driver.set(webDriver);
    }

    public static void quitDriver() {
        if (driver.get() != null) {
            driver.get().quit();
            driver.remove();
        }
    }
}

// Static utility methods
public class TestUtils {
    public static String getTimestamp() {
        return new SimpleDateFormat("yyyyMMdd_HHmmss").format(new Date());
    }

    public static String takeScreenshot(WebDriver driver, String testName) {
        File src = ((TakesScreenshot) driver).getScreenshotAs(OutputType.FILE);
        String path = "screenshots/" + testName + "_" + getTimestamp() + ".png";
        FileUtils.copyFile(src, new File(path));
        return path;
    }
}
```

---

### Q9. What is the difference between `final`, `finally`, and `finalize()`?

**Answer:**

| Keyword | Purpose | Automation Usage |
|---|---|---|
| `final` | Makes variable constant, method un-overridable, class un-inheritable | `final String BASE_URL = "https://app.com";` — constants in config |
| `finally` | Block that always executes after try/catch | Closing database connections, quitting browser |
| `finalize()` | Called by GC before object destruction (deprecated in Java 9+) | Rarely used; not recommended |

```java
public void testWithCleanup() {
    WebDriver driver = null;
    try {
        driver = new ChromeDriver();
        driver.get("https://example.com");
        // test steps
    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        if (driver != null) {
            driver.quit(); // always runs — ensures browser cleanup
        }
    }
}
```

---

### Q10. Explain Streams API. How do you use it in test automation?

**Answer:**

```java
// Filter failed test results
List<TestResult> results = getTestResults();
List<TestResult> failures = results.stream()
    .filter(r -> r.getStatus().equals("FAILED"))
    .sorted(Comparator.comparing(TestResult::getTestName))
    .collect(Collectors.toList());

// Extract all href attributes from links
List<String> allHrefs = driver.findElements(By.tagName("a")).stream()
    .map(e -> e.getAttribute("href"))
    .filter(Objects::nonNull)
    .filter(href -> !href.isEmpty())
    .distinct()
    .collect(Collectors.toList());

// Check if any error messages are displayed
boolean hasErrors = driver.findElements(By.className("error")).stream()
    .anyMatch(WebElement::isDisplayed);

// Convert list of WebElements to text
List<String> menuItems = driver.findElements(By.cssSelector(".menu-item")).stream()
    .map(WebElement::getText)
    .collect(Collectors.toList());
```

---

## 2. Selenium WebDriver

### Q11. Explain the Selenium WebDriver architecture.

**Answer:**

```
Test Script (Java)  →  Selenium Client Library  →  JSON Wire Protocol / W3C WebDriver Protocol
                                                            ↓
                                                    Browser Driver (ChromeDriver/GeckoDriver)
                                                            ↓
                                                    Real Browser (Chrome/Firefox)
```

**Components:**
1. **Selenium Client Libraries** — Java bindings that provide the API (`WebDriver`, `WebElement`, etc.)
2. **JSON Wire Protocol / W3C Protocol** — HTTP-based communication protocol between client and driver
3. **Browser Drivers** — Executable files (chromedriver, geckodriver) that translate commands into browser-native calls
4. **Real Browsers** — Chrome, Firefox, Edge that execute the actions

**How it works:**
1. Test calls `driver.findElement(By.id("login"))` 
2. Client library serializes this into an HTTP POST request
3. Request sent to browser driver server (e.g., `http://localhost:9515/session/{id}/element`)
4. Driver translates to browser-native command
5. Browser executes and returns result
6. Response flows back through the chain

---

### Q12. What are the different types of waits in Selenium? When do you use each?

**Answer:**

| Wait Type | Scope | Usage | Recommendation |
|---|---|---|---|
| **Implicit Wait** | Global — applies to all `findElement` calls | `driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10))` | ⚠️ Avoid — mixes with explicit waits unpredictably |
| **Explicit Wait** | Specific — applies to a single condition | `new WebDriverWait(driver, Duration.ofSeconds(10)).until(...)` | ✅ Preferred — targeted and precise |
| **Fluent Wait** | Specific — with polling interval & exception ignoring | Configure polling frequency, ignore specific exceptions | ✅ Best for dynamic elements |
| **Thread.sleep()** | Hard pause | `Thread.sleep(3000)` | ❌ Never use in production code |

```java
// Explicit Wait — most common
WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(15));
WebElement element = wait.until(
    ExpectedConditions.visibilityOfElementLocated(By.id("dashboard"))
);

// Fluent Wait — for unreliable elements
Wait<WebDriver> fluentWait = new FluentWait<>(driver)
    .withTimeout(Duration.ofSeconds(30))
    .pollingEvery(Duration.ofMillis(500))
    .ignoring(NoSuchElementException.class)
    .ignoring(StaleElementReferenceException.class);

WebElement element = fluentWait.until(d -> d.findElement(By.id("dynamic-element")));

// Custom wait condition
wait.until(driver -> {
    List<WebElement> rows = driver.findElements(By.cssSelector("table tbody tr"));
    return rows.size() > 5; // wait until table has more than 5 rows
});
```

---

### Q13. How do you handle dynamic elements / elements that load asynchronously?

**Answer:**

```java
// 1. Dynamic IDs — use partial attribute matching
// Instead of: By.id("element_12345")
By.cssSelector("[id^='element_']")       // starts with
By.cssSelector("[id$='_submit']")        // ends with
By.cssSelector("[id*='login']")          // contains
By.xpath("//*[contains(@id, 'login')]")

// 2. Wait for AJAX calls to complete
public void waitForAjaxComplete() {
    new WebDriverWait(driver, Duration.ofSeconds(30)).until(d -> {
        JavascriptExecutor js = (JavascriptExecutor) d;
        return (Boolean) js.executeScript("return jQuery.active == 0");
    });
}

// 3. Wait for page load state
public void waitForPageLoad() {
    new WebDriverWait(driver, Duration.ofSeconds(30)).until(d -> {
        return ((JavascriptExecutor) d)
            .executeScript("return document.readyState").equals("complete");
    });
}

// 4. Stale element — retry pattern
public WebElement findElementWithRetry(By locator, int maxRetries) {
    for (int i = 0; i < maxRetries; i++) {
        try {
            WebElement element = driver.findElement(locator);
            element.isDisplayed(); // trigger staleness check
            return element;
        } catch (StaleElementReferenceException e) {
            if (i == maxRetries - 1) throw e;
        }
    }
    throw new NoSuchElementException("Element not found: " + locator);
}
```

---

### Q14. How do you handle frames, windows, and alerts in Selenium?

**Answer:**

```java
// ===== FRAMES / IFRAMES =====
// Switch by index
driver.switchTo().frame(0);

// Switch by name or ID
driver.switchTo().frame("iframe-name");

// Switch by WebElement
WebElement iframe = driver.findElement(By.cssSelector("iframe.payment-frame"));
driver.switchTo().frame(iframe);

// Switch back to main page
driver.switchTo().defaultContent();

// Switch to parent frame (one level up)
driver.switchTo().parentFrame();

// ===== WINDOWS / TABS =====
String mainWindow = driver.getWindowHandle();
Set<String> allWindows = driver.getWindowHandles();

for (String window : allWindows) {
    if (!window.equals(mainWindow)) {
        driver.switchTo().window(window);
        // perform actions in new window
        driver.close(); // close this window
    }
}
driver.switchTo().window(mainWindow); // switch back

// ===== ALERTS =====
// Simple alert
Alert alert = driver.switchTo().alert();
String alertText = alert.getText();
alert.accept();    // click OK
alert.dismiss();   // click Cancel

// Prompt alert
alert.sendKeys("input text");
alert.accept();

// Wait for alert
WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(5));
wait.until(ExpectedConditions.alertIsPresent());
```

---

### Q15. What locator strategies do you use and what is their priority?

**Answer:**

| Priority | Locator | Speed | Reliability | When to Use |
|---|---|---|---|---|
| 1 | `By.id()` | Fastest | Highest | Element has unique, stable ID |
| 2 | `By.name()` | Fast | High | Form elements with name attribute |
| 3 | `By.cssSelector()` | Fast | High | Complex selectors, no ID available |
| 4 | `By.className()` | Fast | Medium | Unique class names |
| 5 | `By.linkText()` | Medium | Medium | Exact link text |
| 6 | `By.partialLinkText()` | Medium | Medium | Partial link text |
| 7 | `By.tagName()` | Fast | Low | Collecting all elements of a type |
| 8 | `By.xpath()` | Slowest | Varies | Complex traversals, text-based, last resort |

```java
// CSS Selector best practices
By.cssSelector("#loginForm input[type='email']")    // child of ID
By.cssSelector("div.card > h2.title")               // direct child
By.cssSelector("ul.menu li:nth-child(3)")           // nth item
By.cssSelector("[data-testid='submit-btn']")        // data attribute ← BEST PRACTICE

// XPath — when CSS can't do it
By.xpath("//button[text()='Submit']")               // exact text match
By.xpath("//div[contains(@class,'active')]//span")  // contains + descendant
By.xpath("//input[@type='text']/following-sibling::span") // sibling traversal
By.xpath("//td[text()='John']/parent::tr//button")  // parent traversal
```

---

### Q16. How do you handle dropdowns, file uploads, and drag-and-drop?

**Answer:**

```java
// ===== DROPDOWNS =====
// Using Select class (for <select> elements)
Select dropdown = new Select(driver.findElement(By.id("country")));
dropdown.selectByVisibleText("India");
dropdown.selectByValue("IN");
dropdown.selectByIndex(2);

// Get all options
List<WebElement> options = dropdown.getOptions();
// Get selected option
String selected = dropdown.getFirstSelectedOption().getText();

// Custom dropdown (non-select)
driver.findElement(By.cssSelector(".dropdown-toggle")).click();
WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(5));
wait.until(ExpectedConditions.visibilityOfElementLocated(By.cssSelector(".dropdown-menu")));
driver.findElement(By.xpath("//li[text()='Option 1']")).click();

// ===== FILE UPLOAD =====
WebElement uploadInput = driver.findElement(By.cssSelector("input[type='file']"));
uploadInput.sendKeys("/path/to/file.pdf"); // absolute path

// ===== DRAG AND DROP =====
Actions actions = new Actions(driver);
WebElement source = driver.findElement(By.id("draggable"));
WebElement target = driver.findElement(By.id("droppable"));
actions.dragAndDrop(source, target).perform();

// Alternative approach if dragAndDrop doesn't work
actions.clickAndHold(source)
       .moveToElement(target)
       .release()
       .perform();
```

---

### Q17. How do you take screenshots in Selenium? At what points do you capture them?

**Answer:**

```java
public class ScreenshotUtil {

    // Full page screenshot
    public static String captureScreenshot(WebDriver driver, String testName) {
        TakesScreenshot ts = (TakesScreenshot) driver;
        File source = ts.getScreenshotAs(OutputType.FILE);
        String destination = "screenshots/" + testName + "_" +
            LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyyMMdd_HHmmss")) + ".png";
        try {
            FileUtils.copyFile(source, new File(destination));
        } catch (IOException e) {
            e.printStackTrace();
        }
        return destination;
    }

    // Element screenshot (Selenium 4)
    public static void captureElementScreenshot(WebElement element, String name) {
        File src = element.getScreenshotAs(OutputType.FILE);
        FileUtils.copyFile(src, new File("screenshots/element_" + name + ".png"));
    }

    // For Extent Reports / Allure — Base64
    public static String getBase64Screenshot(WebDriver driver) {
        return ((TakesScreenshot) driver).getScreenshotAs(OutputType.BASE64);
    }
}

// TestNG Listener to capture on failure
public class TestListener implements ITestListener {
    @Override
    public void onTestFailure(ITestResult result) {
        WebDriver driver = DriverManager.getDriver();
        String path = ScreenshotUtil.captureScreenshot(driver, result.getName());
        // Attach to report
    }
}
```

**When to capture screenshots:**
- ✅ On test failure (mandatory)
- ✅ Before and after critical actions (payments, form submissions)
- ✅ On assertion failures
- ✅ For visual regression comparison
- ❌ Not on every step (performance impact)

---

### Q18. What's new in Selenium 4 compared to Selenium 3?

**Answer:**

| Feature | Selenium 3 | Selenium 4 |
|---|---|---|
| **Protocol** | JSON Wire Protocol | W3C WebDriver Protocol (native) |
| **Driver Manager** | Manual driver setup | Selenium Manager (auto downloads drivers) |
| **Relative Locators** | Not available | `RelativeLocator.with(By.tagName("input")).above(element)` |
| **New Window/Tab** | JavaScript workaround | `driver.switchTo().newWindow(WindowType.TAB)` |
| **Element Screenshots** | Not available | `element.getScreenshotAs(OutputType.FILE)` |
| **Chrome DevTools** | Not available | CDP support via `DevTools` interface |
| **Grid** | Hub + Node architecture | Docker support, fully distributed |

```java
// Relative Locators (Selenium 4)
WebElement emailField = driver.findElement(By.id("email"));
WebElement passwordField = driver.findElement(
    RelativeLocator.with(By.tagName("input")).below(emailField)
);

// Open new tab
driver.switchTo().newWindow(WindowType.TAB);
driver.get("https://second-page.com");

// Chrome DevTools Protocol
DevTools devTools = ((ChromeDriver) driver).getDevTools();
devTools.createSession();
devTools.send(Network.enable(Optional.empty(), Optional.empty(), Optional.empty()));
// Intercept network requests, mock geolocation, simulate throttling, etc.
```

---

## 3. API Testing Concepts

### Q19. What is REST? Explain REST architectural constraints.

**Answer:**

REST (Representational State Transfer) is an architectural style for designing networked applications.

**Six Constraints:**

| Constraint | Description |
|---|---|
| **Client-Server** | Separation of concerns — client handles UI, server handles data/logic |
| **Stateless** | Each request contains all info needed; server doesn't store client context |
| **Cacheable** | Responses must declare themselves cacheable or non-cacheable |
| **Uniform Interface** | Consistent URL patterns, standard HTTP methods, HATEOAS |
| **Layered System** | Client doesn't know if it's connected to end server or intermediary |
| **Code on Demand** (optional) | Server can send executable code to client |

---

### Q20. Explain HTTP methods and their idempotency.

**Answer:**

| Method | Purpose | Idempotent? | Safe? | Request Body |
|---|---|---|---|---|
| `GET` | Retrieve resource | ✅ Yes | ✅ Yes | No |
| `POST` | Create resource | ❌ No | ❌ No | Yes |
| `PUT` | Update/Replace entire resource | ✅ Yes | ❌ No | Yes |
| `PATCH` | Partial update of resource | ❌ No | ❌ No | Yes |
| `DELETE` | Remove resource | ✅ Yes | ❌ No | Optional |
| `HEAD` | Same as GET but no body | ✅ Yes | ✅ Yes | No |
| `OPTIONS` | Describe communication options | ✅ Yes | ✅ Yes | No |

**Idempotent** = Making the same request multiple times produces the same result.
- `PUT /users/1 {name: "John"}` → Always sets name to "John" (idempotent)
- `POST /users {name: "John"}` → Creates a new user each time (not idempotent)

---

### Q21. Explain HTTP status codes and which ones you commonly validate.

**Answer:**

| Range | Category | Common Codes |
|---|---|---|
| **1xx** | Informational | 100 Continue |
| **2xx** | Success | 200 OK, 201 Created, 204 No Content |
| **3xx** | Redirection | 301 Moved Permanently, 302 Found, 304 Not Modified |
| **4xx** | Client Error | 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 405 Method Not Allowed, 409 Conflict, 422 Unprocessable Entity, 429 Too Many Requests |
| **5xx** | Server Error | 500 Internal Server Error, 502 Bad Gateway, 503 Service Unavailable |

**What I validate in API tests:**
```java
// Happy path
given().when().get("/users/1")
    .then().statusCode(200)
           .body("name", equalTo("John"));

// Creation
given().body(newUser).when().post("/users")
    .then().statusCode(201)
           .header("Location", containsString("/users/"));

// Not found
given().when().get("/users/99999")
    .then().statusCode(404)
           .body("error", equalTo("User not found"));

// Unauthorized
given().when().get("/admin/dashboard")  // no auth token
    .then().statusCode(401);

// Validation error
given().body("{\"email\": \"invalid\"}").when().post("/users")
    .then().statusCode(400)
           .body("errors.email", notNullValue());
```

---

### Q22. What is the difference between authentication and authorization? How do you test APIs that require auth?

**Answer:**

- **Authentication** = "Who are you?" → Verifying identity (login, tokens)
- **Authorization** = "What can you do?" → Verifying permissions (admin vs user)

**Common Auth mechanisms and how to test:**

```java
// 1. Basic Auth
given()
    .auth().basic("username", "password")
.when()
    .get("/api/resource")
.then()
    .statusCode(200);

// 2. Bearer Token (JWT)
String token = getAuthToken("admin", "password");
given()
    .header("Authorization", "Bearer " + token)
.when()
    .get("/api/admin/users")
.then()
    .statusCode(200);

// 3. OAuth 2.0
given()
    .auth().oauth2(accessToken)
.when()
    .get("/api/resource")
.then()
    .statusCode(200);

// 4. API Key
given()
    .header("X-API-Key", "my-api-key-123")
.when()
    .get("/api/data")
.then()
    .statusCode(200);

// Test cases for auth:
// ✅ Valid credentials → 200
// ❌ Invalid credentials → 401
// ❌ Expired token → 401
// ❌ Valid user, insufficient permissions → 403
// ❌ Missing auth header → 401
```

---

### Q23. What is the difference between PUT and PATCH?

**Answer:**

| Aspect | PUT | PATCH |
|---|---|---|
| **Purpose** | Replace entire resource | Partially update resource |
| **Required fields** | All fields must be sent | Only changed fields |
| **Missing fields** | Set to null/default | Unchanged |
| **Idempotent** | Yes | No (generally) |

```java
// Original user: { "id": 1, "name": "John", "email": "john@test.com", "age": 30 }

// PUT — replaces everything (if you omit "age", it becomes null)
given()
    .body("{ \"name\": \"John Updated\", \"email\": \"john@test.com\", \"age\": 30 }")
.when()
    .put("/users/1");

// PATCH — updates only specified fields (age stays 30)
given()
    .body("{ \"name\": \"John Updated\" }")
.when()
    .patch("/users/1");
```

---

## 4. REST Assured

### Q24. What is REST Assured? Explain its architecture and the Given-When-Then pattern.

**Answer:**

REST Assured is a Java library for testing REST APIs. It follows the **BDD (Given-When-Then)** pattern:

- **Given** — Pre-conditions: headers, auth, body, query params
- **When** — Action: HTTP method and endpoint
- **Then** — Assertions: status code, response body, headers

```java
import static io.restassured.RestAssured.*;
import static org.hamcrest.Matchers.*;

@Test
public void testGetUser() {
    given()
        .baseUri("https://api.example.com")
        .header("Content-Type", "application/json")
        .header("Authorization", "Bearer " + token)
        .queryParam("fields", "name,email")
        .log().all()                         // log request
    .when()
        .get("/users/1")
    .then()
        .log().all()                         // log response
        .statusCode(200)
        .contentType(ContentType.JSON)
        .body("name", equalTo("John"))
        .body("email", containsString("@"))
        .body("roles", hasSize(2))
        .body("roles", hasItem("admin"))
        .time(lessThan(3000L));              // response time < 3s
}
```

---

### Q25. How do you validate JSON response body using REST Assured?

**Answer:**

```java
// Sample response:
// {
//   "id": 1,
//   "name": "John",
//   "address": { "city": "New York", "zip": "10001" },
//   "phones": ["+1-111-111", "+1-222-222"],
//   "orders": [
//     { "id": 101, "total": 50.00, "status": "delivered" },
//     { "id": 102, "total": 75.50, "status": "pending" }
//   ]
// }

given().when().get("/users/1").then()
    // Simple field
    .body("name", equalTo("John"))

    // Nested object
    .body("address.city", equalTo("New York"))

    // Array element
    .body("phones[0]", equalTo("+1-111-111"))
    .body("phones", hasSize(2))

    // Array of objects — JsonPath
    .body("orders[0].status", equalTo("delivered"))
    .body("orders.findAll { it.total > 60 }.size()", equalTo(1))
    .body("orders.find { it.id == 102 }.status", equalTo("pending"))
    .body("orders.collect { it.total }.sum()", equalTo(125.5f))

    // Check field exists
    .body("$", hasKey("name"))
    .body("address", hasKey("zip"))

    // Null checks
    .body("middleName", nullValue())
    .body("name", notNullValue());
```

---

### Q26. How do you send POST requests with different body types?

**Answer:**

```java
// 1. JSON String
given()
    .contentType(ContentType.JSON)
    .body("{ \"name\": \"John\", \"email\": \"john@test.com\" }")
.when()
    .post("/users");

// 2. Java Map → auto-serialized to JSON
Map<String, Object> user = new HashMap<>();
user.put("name", "John");
user.put("email", "john@test.com");

given()
    .contentType(ContentType.JSON)
    .body(user)
.when()
    .post("/users");

// 3. POJO → auto-serialized (best practice)
public class User {
    private String name;
    private String email;
    // constructors, getters, setters
}

User user = new User("John", "john@test.com");
given()
    .contentType(ContentType.JSON)
    .body(user)
.when()
    .post("/users")
.then()
    .statusCode(201);

// 4. From JSON file
File jsonFile = new File("src/test/resources/testdata/user.json");
given()
    .contentType(ContentType.JSON)
    .body(jsonFile)
.when()
    .post("/users");

// 5. Form data
given()
    .contentType(ContentType.URLENC)
    .formParam("username", "john")
    .formParam("password", "secret")
.when()
    .post("/login");

// 6. Multipart (file upload)
given()
    .multiPart("file", new File("report.pdf"))
    .multiPart("description", "Monthly Report")
.when()
    .post("/upload");
```

---

### Q27. How do you extract values from responses and chain API calls?

**Answer:**

```java
// Extract single value
String userId = given()
    .contentType(ContentType.JSON)
    .body(newUser)
.when()
    .post("/users")
.then()
    .statusCode(201)
    .extract().path("id");

// Use extracted value in next request
given()
    .pathParam("id", userId)
.when()
    .get("/users/{id}")
.then()
    .statusCode(200)
    .body("name", equalTo("John"));

// Extract entire response
Response response = given().when().get("/users/1");
String name = response.jsonPath().getString("name");
int statusCode = response.getStatusCode();
String contentType = response.getContentType();
long responseTime = response.getTime();
Headers headers = response.getHeaders();

// Chaining — Create user → Get user → Update user → Delete user
@Test
public void testUserCRUDFlow() {
    // CREATE
    String userId = given().body(newUser).when().post("/users")
        .then().statusCode(201).extract().path("id");

    // READ
    given().when().get("/users/" + userId)
        .then().statusCode(200).body("name", equalTo("John"));

    // UPDATE
    given().body("{\"name\": \"John Updated\"}").when().put("/users/" + userId)
        .then().statusCode(200);

    // DELETE
    given().when().delete("/users/" + userId)
        .then().statusCode(204);

    // VERIFY DELETION
    given().when().get("/users/" + userId)
        .then().statusCode(404);
}
```

---

### Q28. How do you handle response deserialization (JSON to POJO)?

**Answer:**

```java
// POJO class
@Data // Lombok
public class UserResponse {
    private int id;
    private String name;
    private String email;
    private Address address;
    private List<String> roles;
}

@Data
public class Address {
    private String city;
    private String zipCode;
}

// Deserialize response to POJO
UserResponse user = given()
    .when()
        .get("/users/1")
    .then()
        .statusCode(200)
        .extract()
        .as(UserResponse.class);

// Now use Java object — type-safe assertions
assertEquals("John", user.getName());
assertEquals("New York", user.getAddress().getCity());
assertTrue(user.getRoles().contains("admin"));

// Deserialize list of objects
List<UserResponse> users = given()
    .when()
        .get("/users")
    .then()
        .extract()
        .jsonPath()
        .getList(".", UserResponse.class);

assertEquals(10, users.size());
```

---

### Q29. How do you set up request/response specifications in REST Assured?

**Answer:**

```java
public class ApiTestBase {
    protected RequestSpecification requestSpec;
    protected ResponseSpecification responseSpec;

    @BeforeClass
    public void setup() {
        // Request Specification — common setup for all requests
        requestSpec = new RequestSpecBuilder()
            .setBaseUri("https://api.example.com")
            .setBasePath("/v1")
            .setContentType(ContentType.JSON)
            .addHeader("Authorization", "Bearer " + getToken())
            .addFilter(new RequestLoggingFilter())   // log all requests
            .addFilter(new ResponseLoggingFilter())  // log all responses
            .build();

        // Response Specification — common assertions for all responses
        responseSpec = new ResponseSpecBuilder()
            .expectContentType(ContentType.JSON)
            .expectResponseTime(lessThan(5000L))
            .expectHeader("X-Request-Id", notNullValue())
            .build();
    }

    @Test
    public void testGetUser() {
        given()
            .spec(requestSpec)                   // apply request spec
            .pathParam("id", 1)
        .when()
            .get("/users/{id}")
        .then()
            .spec(responseSpec)                  // apply response spec
            .statusCode(200)
            .body("name", equalTo("John"));
    }
}
```

---

### Q30. How do you validate JSON Schema in REST Assured?

**Answer:**

```java
// Add dependency: io.rest-assured:json-schema-validator

// JSON Schema file: src/test/resources/schemas/user-schema.json
// {
//   "$schema": "http://json-schema.org/draft-07/schema#",
//   "type": "object",
//   "required": ["id", "name", "email"],
//   "properties": {
//     "id": { "type": "integer" },
//     "name": { "type": "string", "minLength": 1 },
//     "email": { "type": "string", "format": "email" },
//     "roles": {
//       "type": "array",
//       "items": { "type": "string" }
//     }
//   },
//   "additionalProperties": false
// }

import static io.restassured.module.jsv.JsonSchemaValidator.*;

@Test
public void testUserResponseMatchesSchema() {
    given()
    .when()
        .get("/users/1")
    .then()
        .statusCode(200)
        .body(matchesJsonSchemaInClasspath("schemas/user-schema.json"));
}
```

**Why schema validation matters:**
- Catches unexpected field additions/removals
- Validates data types (string vs number)
- Ensures required fields are present
- Protects against breaking API changes

---

## 5. Test Automation Framework Design

### Q31. Describe the automation framework you have designed/worked with. What design patterns did you use?

**Answer:**

> **Framework: Hybrid Framework (Page Object Model + Data-Driven + Keyword-Driven)**

```
project-root/
├── src/main/java/
│   ├── pages/                    # Page Object classes
│   │   ├── BasePage.java
│   │   ├── LoginPage.java
│   │   └── DashboardPage.java
│   ├── api/                      # API wrapper classes
│   │   ├── BaseApi.java
│   │   └── UserApi.java
│   ├── utils/                    # Utility classes
│   │   ├── DriverManager.java    # WebDriver lifecycle (Singleton/Factory)
│   │   ├── ConfigReader.java     # Properties file reader
│   │   ├── ExcelUtils.java       # Excel data reader
│   │   ├── WaitUtils.java        # Custom wait methods
│   │   └── ReportManager.java    # Extent Reports setup
│   └── constants/                # Constants and enums
│       └── AppConstants.java
├── src/test/java/
│   ├── tests/
│   │   ├── BaseTest.java         # @BeforeMethod, @AfterMethod
│   │   ├── LoginTest.java
│   │   └── api/
│   │       └── UserApiTest.java
│   ├── listeners/
│   │   └── TestListener.java     # ITestListener for reporting
│   └── dataproviders/
│       └── LoginDataProvider.java
├── src/test/resources/
│   ├── config/
│   │   ├── config.properties
│   │   └── config-qa.properties
│   ├── testdata/
│   │   ├── login-data.xlsx
│   │   └── api-payloads/
│   └── schemas/                  # JSON schemas for API validation
├── testng.xml                    # Test suite configuration
├── pom.xml                       # Maven dependencies
└── Jenkinsfile                   # CI/CD pipeline
```

**Design Patterns Used:**

| Pattern | Where Used | Purpose |
|---|---|---|
| **Page Object Model** | `pages/` | Separates locators and actions from tests |
| **Factory Pattern** | `DriverManager.java` | Creates browser instances based on config |
| **Singleton Pattern** | `ConfigReader.java` | Single instance for configuration |
| **Builder Pattern** | API request construction | Fluent API call building |
| **Strategy Pattern** | Multiple browser support | Switch browser strategy at runtime |
| **Data Provider** | TestNG DataProvider | Externalize test data |

---

### Q32. Explain Page Object Model (POM) in detail. How does Page Factory enhance it?

**Answer:**

```java
// ===== BASE PAGE =====
public abstract class BasePage {
    protected WebDriver driver;
    protected WebDriverWait wait;

    public BasePage(WebDriver driver) {
        this.driver = driver;
        this.wait = new WebDriverWait(driver, Duration.ofSeconds(15));
        PageFactory.initElements(driver, this);
    }

    protected void click(WebElement element) {
        wait.until(ExpectedConditions.elementToBeClickable(element)).click();
    }

    protected void type(WebElement element, String text) {
        wait.until(ExpectedConditions.visibilityOf(element));
        element.clear();
        element.sendKeys(text);
    }

    protected String getText(WebElement element) {
        return wait.until(ExpectedConditions.visibilityOf(element)).getText();
    }

    protected boolean isDisplayed(WebElement element) {
        try {
            return element.isDisplayed();
        } catch (NoSuchElementException e) {
            return false;
        }
    }
}

// ===== LOGIN PAGE =====
public class LoginPage extends BasePage {
    @FindBy(id = "username")
    private WebElement usernameField;

    @FindBy(id = "password")
    private WebElement passwordField;

    @FindBy(css = "button[type='submit']")
    private WebElement loginButton;

    @FindBy(css = ".error-message")
    private WebElement errorMessage;

    public LoginPage(WebDriver driver) {
        super(driver);
    }

    // Page actions return next page object (method chaining)
    public DashboardPage loginAs(String username, String password) {
        type(usernameField, username);
        type(passwordField, password);
        click(loginButton);
        return new DashboardPage(driver);
    }

    public LoginPage loginExpectingError(String username, String password) {
        type(usernameField, username);
        type(passwordField, password);
        click(loginButton);
        return this; // stays on same page
    }

    public String getErrorMessage() {
        return getText(errorMessage);
    }
}

// ===== TEST CLASS =====
public class LoginTest extends BaseTest {
    @Test
    public void testSuccessfulLogin() {
        DashboardPage dashboard = new LoginPage(driver)
            .loginAs("admin", "password123");
        assertEquals(dashboard.getWelcomeMessage(), "Welcome, Admin!");
    }

    @Test
    public void testInvalidLogin() {
        LoginPage loginPage = new LoginPage(driver)
            .loginExpectingError("invalid", "wrong");
        assertEquals(loginPage.getErrorMessage(), "Invalid credentials");
    }
}
```

**Page Factory advantages:**
- `@FindBy` — Cleaner element declaration
- Lazy initialization — Elements are found when used, not when declared
- `@CacheLookup` — Caches element reference for static elements (use carefully)

---

### Q33. How do you handle test data management in your framework?

**Answer:**

```java
// 1. Properties files — Environment config
// config-qa.properties
// base.url=https://qa.example.com
// api.url=https://api-qa.example.com

public class ConfigReader {
    private static Properties props;
    private static final String ENV = System.getProperty("env", "qa");

    static {
        try {
            props = new Properties();
            props.load(new FileInputStream("config/config-" + ENV + ".properties"));
        } catch (IOException e) {
            throw new RuntimeException("Config file not found for env: " + ENV);
        }
    }

    public static String get(String key) {
        return props.getProperty(key);
    }
}

// 2. TestNG DataProvider — Parameterized tests
@DataProvider(name = "loginData")
public Object[][] getLoginData() {
    return new Object[][] {
        {"admin", "admin123", true},
        {"user", "user123", true},
        {"invalid", "wrong", false},
        {"", "password", false},
        {"admin", "", false}
    };
}

@Test(dataProvider = "loginData")
public void testLogin(String username, String password, boolean shouldPass) {
    LoginPage loginPage = new LoginPage(driver);
    if (shouldPass) {
        DashboardPage dashboard = loginPage.loginAs(username, password);
        assertTrue(dashboard.isLoaded());
    } else {
        loginPage.loginExpectingError(username, password);
        assertTrue(loginPage.getErrorMessage().length() > 0);
    }
}

// 3. Excel data — Apache POI
@DataProvider(name = "excelData")
public Object[][] getExcelData() {
    return ExcelUtils.readData("testdata/registration.xlsx", "Sheet1");
}

// 4. JSON test data
public class TestDataReader {
    public static <T> T readJson(String filePath, Class<T> clazz) {
        ObjectMapper mapper = new ObjectMapper();
        return mapper.readValue(new File(filePath), clazz);
    }
}
```

---

### Q34. How do you implement parallel test execution?

**Answer:**

```xml
<!-- testng.xml — Parallel execution -->
<suite name="Regression Suite" parallel="methods" thread-count="4">
    <test name="Chrome Tests">
        <parameter name="browser" value="chrome"/>
        <classes>
            <class name="tests.LoginTest"/>
            <class name="tests.SearchTest"/>
        </classes>
    </test>
</suite>

<!-- Cross-browser parallel -->
<suite name="Cross Browser" parallel="tests" thread-count="3">
    <test name="Chrome">
        <parameter name="browser" value="chrome"/>
        <classes><class name="tests.SmokeTests"/></classes>
    </test>
    <test name="Firefox">
        <parameter name="browser" value="firefox"/>
        <classes><class name="tests.SmokeTests"/></classes>
    </test>
    <test name="Edge">
        <parameter name="browser" value="edge"/>
        <classes><class name="tests.SmokeTests"/></classes>
    </test>
</suite>
```

```java
// Thread-safe driver management
public class DriverManager {
    private static ThreadLocal<WebDriver> driverThread = new ThreadLocal<>();

    public static void initDriver(String browser) {
        WebDriver driver;
        switch (browser.toLowerCase()) {
            case "chrome":
                driver = new ChromeDriver(getChromeOptions());
                break;
            case "firefox":
                driver = new FirefoxDriver(getFirefoxOptions());
                break;
            default:
                driver = new ChromeDriver();
        }
        driverThread.set(driver);
    }

    public static WebDriver getDriver() {
        return driverThread.get();
    }

    public static void quitDriver() {
        if (driverThread.get() != null) {
            driverThread.get().quit();
            driverThread.remove(); // prevent memory leaks
        }
    }
}
```

---

### Q35. How do you generate test reports in your framework?

**Answer:**

```java
// ===== EXTENT REPORTS =====
public class ReportManager {
    private static ExtentReports extent;
    private static ThreadLocal<ExtentTest> test = new ThreadLocal<>();

    public static ExtentReports getInstance() {
        if (extent == null) {
            ExtentSparkReporter spark = new ExtentSparkReporter("reports/extent-report.html");
            spark.config().setTheme(Theme.DARK);
            spark.config().setDocumentTitle("Automation Report");
            spark.config().setReportName("Regression Suite");

            extent = new ExtentReports();
            extent.attachReporter(spark);
            extent.setSystemInfo("Environment", ConfigReader.get("env"));
            extent.setSystemInfo("Browser", ConfigReader.get("browser"));
            extent.setSystemInfo("OS", System.getProperty("os.name"));
        }
        return extent;
    }

    public static void startTest(String testName, String description) {
        ExtentTest extentTest = getInstance().createTest(testName, description);
        test.set(extentTest);
    }

    public static void logPass(String message) {
        test.get().pass(message);
    }

    public static void logFail(String message, String screenshotPath) {
        test.get().fail(message)
            .addScreenCaptureFromPath(screenshotPath);
    }
}

// ===== LISTENER =====
public class TestListener implements ITestListener {
    @Override
    public void onTestStart(ITestResult result) {
        ReportManager.startTest(result.getMethod().getMethodName(),
            result.getMethod().getDescription());
    }

    @Override
    public void onTestSuccess(ITestResult result) {
        ReportManager.logPass("Test passed");
    }

    @Override
    public void onTestFailure(ITestResult result) {
        String screenshot = ScreenshotUtil.capture(
            DriverManager.getDriver(), result.getName());
        ReportManager.logFail(result.getThrowable().getMessage(), screenshot);
    }

    @Override
    public void onFinish(ITestContext context) {
        ReportManager.getInstance().flush();
    }
}
```

---

## 6. SQL for Testers

### Q36. What SQL queries do you commonly use in test automation?

**Answer:**

```sql
-- 1. Verify data after API/UI operations
SELECT * FROM users WHERE email = 'john@test.com';

-- 2. Validate record count
SELECT COUNT(*) FROM orders WHERE status = 'PENDING';

-- 3. Join tables for data verification
SELECT u.name, o.order_id, o.total, o.status
FROM users u
INNER JOIN orders o ON u.id = o.user_id
WHERE u.email = 'john@test.com'
ORDER BY o.created_at DESC;

-- 4. Check for duplicates
SELECT email, COUNT(*) as cnt
FROM users
GROUP BY email
HAVING COUNT(*) > 1;

-- 5. Subquery — find users with no orders
SELECT name, email FROM users
WHERE id NOT IN (SELECT DISTINCT user_id FROM orders);

-- 6. Date-based queries
SELECT * FROM audit_log
WHERE created_at >= CURRENT_DATE - INTERVAL '7' DAY
AND action = 'LOGIN_FAILED';

-- 7. Update test data (setup/teardown)
UPDATE users SET status = 'ACTIVE' WHERE email = 'testuser@test.com';
DELETE FROM sessions WHERE user_id = 999;

-- 8. Aggregate functions
SELECT 
    status,
    COUNT(*) as count,
    SUM(total) as total_amount,
    AVG(total) as avg_amount,
    MAX(total) as max_amount
FROM orders
GROUP BY status;
```

---

### Q37. How do you integrate database validation in your automation framework?

**Answer:**

```java
public class DatabaseUtils {
    private static Connection getConnection() throws SQLException {
        return DriverManager.getConnection(
            ConfigReader.get("db.url"),
            ConfigReader.get("db.username"),
            ConfigReader.get("db.password")
        );
    }

    // Execute SELECT query
    public static List<Map<String, Object>> executeQuery(String query, Object... params) {
        List<Map<String, Object>> results = new ArrayList<>();
        try (Connection conn = getConnection();
             PreparedStatement stmt = conn.prepareStatement(query)) {

            for (int i = 0; i < params.length; i++) {
                stmt.setObject(i + 1, params[i]);
            }

            ResultSet rs = stmt.executeQuery();
            ResultSetMetaData meta = rs.getMetaData();
            int columnCount = meta.getColumnCount();

            while (rs.next()) {
                Map<String, Object> row = new LinkedHashMap<>();
                for (int i = 1; i <= columnCount; i++) {
                    row.put(meta.getColumnName(i), rs.getObject(i));
                }
                results.add(row);
            }
        } catch (SQLException e) {
            throw new RuntimeException("Query failed: " + query, e);
        }
        return results;
    }

    // Execute INSERT/UPDATE/DELETE
    public static int executeUpdate(String query, Object... params) {
        try (Connection conn = getConnection();
             PreparedStatement stmt = conn.prepareStatement(query)) {

            for (int i = 0; i < params.length; i++) {
                stmt.setObject(i + 1, params[i]);
            }
            return stmt.executeUpdate();
        } catch (SQLException e) {
            throw new RuntimeException("Update failed: " + query, e);
        }
    }
}

// Usage in test
@Test
public void testUserRegistration() {
    // UI: Register user
    registrationPage.register("John", "john@test.com", "password123");

    // DB: Verify user was created
    List<Map<String, Object>> result = DatabaseUtils.executeQuery(
        "SELECT * FROM users WHERE email = ?", "john@test.com");

    assertEquals(1, result.size());
    assertEquals("John", result.get(0).get("name"));
    assertEquals("ACTIVE", result.get(0).get("status"));
}
```

---

### Q38. Explain different types of JOINs with examples relevant to testing.

**Answer:**

| JOIN Type | Returns | Use Case |
|---|---|---|
| `INNER JOIN` | Only matching rows from both tables | "Show me users who have placed orders" |
| `LEFT JOIN` | All rows from left + matching from right | "Show me all users and their orders (including users with no orders)" |
| `RIGHT JOIN` | All rows from right + matching from left | "Show me all orders and their users (including orphan orders)" |
| `FULL OUTER JOIN` | All rows from both tables | "Show me everything — all users and all orders" |
| `CROSS JOIN` | Cartesian product | Generating test combinations |

```sql
-- Verify user-order relationship after order placement
-- INNER JOIN: Users who placed orders
SELECT u.name, o.order_id, o.total
FROM users u
INNER JOIN orders o ON u.id = o.user_id
WHERE u.email = 'john@test.com';

-- LEFT JOIN: Find users without orders (data quality check)
SELECT u.name, u.email, o.order_id
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE o.order_id IS NULL;

-- Self JOIN: Find duplicate records
SELECT a.id, a.email, b.id AS duplicate_id
FROM users a
INNER JOIN users b ON a.email = b.email AND a.id < b.id;
```

---

## 7. Agile Methodology

### Q39. What is Agile? Explain the Scrum framework and your role in it.

**Answer:**

**Agile** is an iterative approach to software development that delivers value in small increments.

**Scrum Framework:**

```
Product Backlog → Sprint Planning → Sprint (2 weeks) → Sprint Review → Retrospective
                        ↑                                      |
                        |__________ Daily Stand-ups ____________|
```

**My role as an Automation Test Engineer in Scrum:**

| Ceremony | My Contribution |
|---|---|
| **Sprint Planning** | Estimate testing effort, identify what can be automated, raise test dependencies |
| **Daily Stand-up** | Share progress on automation, blockers (env issues, flaky tests), what I'll work on today |
| **Sprint Review** | Demo automation coverage, test results dashboard, defect metrics |
| **Retrospective** | Suggest process improvements, discuss test failures, propose automation ROI |
| **Backlog Refinement** | Review acceptance criteria, suggest testability improvements, identify edge cases |

**Definition of Done (DoD) — what I ensure:**
- ✅ All acceptance criteria have test cases
- ✅ Unit tests pass (by developers)
- ✅ Automation scripts written for critical paths
- ✅ Regression suite passes
- ✅ No critical/blocker defects open
- ✅ API contract tests pass

---

### Q40. How do you decide what to automate and what to test manually?

**Answer:**

**Automate:**
- ✅ Regression tests (repeated every sprint)
- ✅ Smoke tests / sanity checks
- ✅ Data-driven tests (same flow, many data combinations)
- ✅ API tests (fast, reliable, high ROI)
- ✅ Cross-browser compatibility tests
- ✅ Performance/load tests

**Keep Manual:**
- 🔍 Exploratory testing
- 🆕 New features (until stable)
- 👁️ Visual/UX testing (subjective)
- 🔒 Complex one-time security audits
- 📱 Device-specific usability testing
- 💥 Ad-hoc testing

**Automation Test Pyramid:**
```
           /  UI/E2E Tests  \      ← Fewest (slow, expensive, brittle)
          /   (Selenium)     \
         /--------------------\
        /  Integration Tests   \   ← Moderate (API, service tests)
       /    (REST Assured)      \
      /--------------------------\
     /      Unit Tests            \  ← Most (fast, reliable, cheap)
    /    (JUnit/TestNG - Devs)     \
   /________________________________\
```

---

### Q41. What is your approach to testing in a Sprint?

**Answer:**

```
Day 1-2: Sprint Planning
├── Review stories and acceptance criteria
├── Write test cases (manual + identify automation candidates)
├── Set up test data and environment
└── Start writing automation scripts for previous sprint stories

Day 3-7: Execution
├── Execute manual tests on new features
├── Write automation scripts in parallel
├── Run API tests as features are deployed to QA
├── Report defects and retest fixes
└── Update regression suite

Day 8-9: Regression & Stabilization
├── Run full regression suite
├── Analyze failures — genuine bugs vs flaky tests
├── Fix flaky tests
├── Performance testing if applicable
└── Update test reports and metrics

Day 10: Sprint Closure
├── Sprint review — present test metrics
├── Ensure all critical bugs are resolved
├── Update automation framework documentation
└── Retrospective — share learnings
```

---

## 8. Testing Fundamentals

### Q42. Explain the different types of testing you perform.

**Answer:**

| Test Type | Purpose | When | Tools |
|---|---|---|---|
| **Smoke Testing** | Quick health check — does the build work? | After every deployment | Selenium, API calls |
| **Functional Testing** | Does the feature work as per requirements? | During sprint | Selenium + TestNG |
| **Regression Testing** | Did new changes break existing features? | Before release | Automated suite |
| **Integration Testing** | Do modules work together? | After feature completion | REST Assured + Selenium |
| **End-to-End Testing** | Full user workflow across systems | Before release | Selenium + API + DB |
| **API Testing** | Validate backend services independently | As APIs are ready | REST Assured |
| **Cross-Browser Testing** | UI consistency across browsers | Before release | Selenium Grid |
| **Performance Testing** | Speed, scalability, stability | Pre-production | JMeter, Gatling |

---

### Q43. What is the difference between verification and validation?

**Answer:**

| Aspect | Verification | Validation |
|---|---|---|
| **Question** | "Are we building the product right?" | "Are we building the right product?" |
| **Focus** | Process and documentation | Actual product behavior |
| **Methods** | Reviews, inspections, walkthroughs | Testing (functional, UAT, E2E) |
| **Phase** | Early (requirements, design) | Later (after implementation) |
| **Example** | Reviewing API specs match requirements | Testing the API actually returns correct data |

---

### Q44. What is a Test Strategy vs. a Test Plan?

**Answer:**

| Aspect | Test Strategy | Test Plan |
|---|---|---|
| **Level** | Organization/project-wide | Specific to a release/sprint |
| **Who creates** | Test Lead / QA Manager | QA Engineers |
| **Content** | Overall approach, tools, environments | Specific test cases, schedule, resources |
| **Changes** | Rarely changes | Updated every sprint/release |
| **Example** | "We use Selenium for UI, REST Assured for API, Jenkins for CI" | "Sprint 15: Test login flow, 20 test cases, 3 automation scripts, complete by Friday" |

---

## 9. Defect Management & Reporting

### Q45. How do you write an effective bug report?

**Answer:**

```
BUG REPORT TEMPLATE:
────────────────────
Title: [Module] Clear, concise description of the defect

Severity: Critical / Major / Minor / Trivial
Priority: P1 (Immediate) / P2 (High) / P3 (Medium) / P4 (Low)

Environment:
- Browser: Chrome 120.0
- OS: Windows 11
- Environment: QA (https://qa.example.com)
- Build: v2.5.3-rc1

Steps to Reproduce:
1. Navigate to https://qa.example.com/login
2. Enter username: "admin@test.com"
3. Enter password: "validPassword123"
4. Click "Login" button
5. Navigate to Settings → Profile
6. Change email to "new-email@test.com"
7. Click "Save"

Expected Result:
- Success message "Profile updated" should appear
- Email should be updated in the database
- Confirmation email sent to new address

Actual Result:
- 500 Internal Server Error displayed
- Email not updated in database
- Console shows: "NullPointerException at UserService.java:145"

Attachments:
- Screenshot of error page
- Browser console logs
- Network request/response (HAR file)
- API response showing 500 error

Additional Notes:
- Reproducible 100% of the time
- Works correctly with shorter email addresses (< 20 chars)
- Possible root cause: email validation regex failing
```

---

### Q46. Explain Severity vs Priority with examples.

**Answer:**

| | High Priority | Low Priority |
|---|---|---|
| **High Severity** | App crashes on login (P1, Critical) | App crashes on rarely-used admin page (P3, Critical) |
| **Low Severity** | Company logo misaligned on home page (P2, Minor) | Typo on "About Us" page (P4, Trivial) |

**Key insight:** Priority is a business decision. Severity is a technical assessment.

- A typo in the company name on the homepage is **Low Severity** (cosmetic) but **High Priority** (brand impact).
- A crash in an edge-case admin feature is **High Severity** (crash) but **Low Priority** (rarely used, few users affected).

---

## 10. CI/CD & DevOps for QA

### Q47. How do you integrate automation tests in CI/CD pipelines?

**Answer:**

```groovy
// Jenkinsfile
pipeline {
    agent any

    environment {
        BROWSER = 'chrome'
        ENV = 'qa'
    }

    stages {
        stage('Build') {
            steps {
                sh 'mvn clean compile'
            }
        }

        stage('Unit Tests') {
            steps {
                sh 'mvn test -Dgroups=unit'
            }
        }

        stage('API Tests') {
            steps {
                sh 'mvn test -Dgroups=api -Denv=${ENV}'
            }
            post {
                always {
                    publishHTML(target: [
                        reportDir: 'reports',
                        reportFiles: 'api-report.html',
                        reportName: 'API Test Report'
                    ])
                }
            }
        }

        stage('UI Smoke Tests') {
            steps {
                sh 'mvn test -Dgroups=smoke -Dbrowser=${BROWSER} -Denv=${ENV}'
            }
        }

        stage('Regression Tests') {
            when { branch 'release/*' }
            steps {
                sh 'mvn test -DsuiteXml=regression-suite.xml -Dbrowser=${BROWSER}'
            }
        }
    }

    post {
        always {
            // Archive test results
            junit 'target/surefire-reports/*.xml'
            // Publish Extent Report
            publishHTML(target: [
                reportDir: 'reports',
                reportFiles: 'extent-report.html',
                reportName: 'Automation Report'
            ])
        }
        failure {
            // Send Slack notification on failure
            slackSend(color: 'danger',
                message: "❌ ${env.JOB_NAME} #${env.BUILD_NUMBER} FAILED")
        }
    }
}
```

---

### Q48. How do you handle flaky tests?

**Answer:**

**Identifying flaky tests:**
- Test passes/fails inconsistently without code changes
- Common causes: timing issues, test data dependencies, environment instability

**Strategies:**

```java
// 1. Retry mechanism
@Test(retryAnalyzer = RetryAnalyzer.class)
public void flakyTest() { ... }

public class RetryAnalyzer implements IRetryAnalyzer {
    private int count = 0;
    private static final int MAX_RETRY = 2;

    @Override
    public boolean retry(ITestResult result) {
        if (count < MAX_RETRY) {
            count++;
            return true; // retry
        }
        return false;
    }
}

// 2. Proper waits (NEVER Thread.sleep)
WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
wait.until(ExpectedConditions.elementToBeClickable(button));

// 3. Test data isolation — each test creates its own data
@BeforeMethod
public void setupTestData() {
    testUser = apiClient.createUser(generateUniqueUser());
}

@AfterMethod
public void cleanupTestData() {
    apiClient.deleteUser(testUser.getId());
}

// 4. Quarantine flaky tests
// testng.xml — separate suite for quarantined tests
// <test name="Quarantine">
//     <groups><run><include name="quarantine"/></run></groups>
// </test>
```

---

## 11. Scenario-Based / Problem-Solving Questions

### Q49. How would you automate testing for a login page?

**Answer:**

```
Test Scenarios for Login Page:
──────────────────────────────
POSITIVE TESTS:
├── TC01: Valid username and valid password → successful login
├── TC02: Login with "Remember Me" checked → session persists
├── TC03: Login and verify redirect to dashboard
└── TC04: Login with different valid user roles (admin, user, viewer)

NEGATIVE TESTS:
├── TC05: Invalid username, valid password → error message
├── TC06: Valid username, invalid password → error message  
├── TC07: Both fields empty → validation message
├── TC08: Username empty, password filled → validation
├── TC09: Username filled, password empty → validation
├── TC10: SQL injection in username: ' OR 1=1 --
├── TC11: XSS in username: <script>alert('xss')</script>
├── TC12: Exceed max length in fields
└── TC13: Special characters in password

SECURITY TESTS:
├── TC14: Password field is masked (type="password")
├── TC15: Credentials not visible in URL (POST not GET)
├── TC16: Account lockout after N failed attempts
├── TC17: Session timeout after inactivity
└── TC18: Cannot login with expired credentials

UI/UX TESTS:
├── TC19: Tab order: username → password → login button
├── TC20: Enter key submits the form
├── TC21: Error messages are user-friendly
└── TC22: Responsive layout on mobile/tablet

API TESTS (Backend):
├── TC23: POST /auth/login with valid credentials → 200 + token
├── TC24: POST /auth/login with invalid creds → 401
├── TC25: POST /auth/login with missing fields → 400
└── TC26: Token expiry and refresh mechanism
```

---

### Q50. You find that 30% of your automated tests are flaky. What do you do?

**Answer:**

**Step 1: Analyze and Categorize**
```
Run the suite 5 times → identify tests that inconsistently fail
Categorize failures:
├── Timing issues (40%) → Add proper waits
├── Test data dependency (25%) → Isolate test data
├── Environment instability (20%) → Coordinate with DevOps
├── Locator fragility (10%) → Use stable locators (data-testid)
└── Test order dependency (5%) → Make tests independent
```

**Step 2: Immediate Actions**
- Quarantine flaky tests into a separate suite
- Keep the main regression suite stable and trustworthy
- Add retry mechanism (max 2 retries) as a temporary fix

**Step 3: Root Cause Fix**
- Replace `Thread.sleep()` with explicit waits
- Create test data per test (API setup) instead of shared data
- Use `data-testid` attributes for locators
- Run tests in random order to find dependencies

**Step 4: Prevention**
- Code review for new tests (check for anti-patterns)
- Dashboard showing flaky test trends
- Fail CI build if flaky rate exceeds threshold
- Regular maintenance sprints for test code

---

### Q51. How do you test an e-commerce checkout flow end-to-end?

**Answer:**

```
E2E Checkout Test Flow:
───────────────────────
1. API Setup (fast data preparation):
   ├── Create test user via API
   ├── Add items to cart via API
   └── Apply coupon code via API

2. UI Verification (critical user journey):
   ├── Login as test user
   ├── Verify cart items and totals
   ├── Enter shipping address
   ├── Select shipping method
   ├── Enter payment details (test card)
   ├── Review order summary
   └── Place order

3. Post-Order Verification:
   ├── UI: Order confirmation page — order number displayed
   ├── API: GET /orders/{id} → status = "PLACED"
   ├── DB: Verify order record, items, totals in database
   ├── DB: Verify inventory decremented
   └── API/Email: Verify confirmation email triggered

4. Teardown:
   ├── Cancel test order via API
   └── Restore inventory
```

```java
@Test(description = "E2E: Complete checkout with valid payment")
public void testCompleteCheckout() {
    // SETUP via API (fast)
    String userId = apiClient.createUser(testUser);
    apiClient.addToCart(userId, "PROD-001", 2);
    apiClient.addToCart(userId, "PROD-002", 1);

    // UI FLOW
    DashboardPage dashboard = new LoginPage(driver).loginAs(testUser);
    CartPage cart = dashboard.goToCart();
    assertEquals(cart.getItemCount(), 3);

    CheckoutPage checkout = cart.proceedToCheckout();
    checkout.enterShippingAddress(testAddress);
    checkout.selectShippingMethod("Express");

    PaymentPage payment = checkout.continueToPayment();
    payment.enterCardDetails("4111111111111111", "12/28", "123");

    OrderConfirmationPage confirmation = payment.placeOrder();
    String orderId = confirmation.getOrderNumber();
    assertNotNull(orderId);
    assertTrue(confirmation.getSuccessMessage().contains("Order placed successfully"));

    // DB VERIFICATION
    Map<String, Object> order = DatabaseUtils.executeQuery(
        "SELECT * FROM orders WHERE order_id = ?", orderId).get(0);
    assertEquals("PLACED", order.get("status"));
    assertEquals(new BigDecimal("159.97"), order.get("total"));
}
```

---

### Q52. How would you design API tests for a User Management microservice?

**Answer:**

```java
public class UserApiTests extends ApiBaseTest {

    // ===== CREATE USER =====
    @Test
    public void testCreateUser_Success() {
        User newUser = User.builder().name("John").email("john@test.com").build();

        Response response = given().spec(requestSpec)
            .body(newUser)
        .when()
            .post("/users")
        .then()
            .statusCode(201)
            .body("id", notNullValue())
            .body("name", equalTo("John"))
            .body("email", equalTo("john@test.com"))
            .body("createdAt", notNullValue())
            .extract().response();

        // Verify in database
        String id = response.path("id");
        verifyUserInDB(id, "John", "john@test.com");
    }

    @Test
    public void testCreateUser_DuplicateEmail_Returns409() {
        User user = createUserViaAPI("john@test.com");

        given().spec(requestSpec)
            .body(User.builder().name("Another John").email("john@test.com").build())
        .when()
            .post("/users")
        .then()
            .statusCode(409)
            .body("error", containsString("already exists"));
    }

    @Test
    public void testCreateUser_InvalidEmail_Returns400() {
        given().spec(requestSpec)
            .body("{\"name\": \"John\", \"email\": \"not-an-email\"}")
        .when()
            .post("/users")
        .then()
            .statusCode(400)
            .body("errors.email", equalTo("Invalid email format"));
    }

    @Test
    public void testCreateUser_MissingRequiredFields_Returns400() {
        given().spec(requestSpec)
            .body("{}")
        .when()
            .post("/users")
        .then()
            .statusCode(400)
            .body("errors", hasKey("name"))
            .body("errors", hasKey("email"));
    }

    // ===== GET USER =====
    @Test
    public void testGetUser_NotFound_Returns404() {
        given().spec(requestSpec)
        .when()
            .get("/users/non-existent-id")
        .then()
            .statusCode(404)
            .body("error", equalTo("User not found"));
    }

    // ===== PAGINATION =====
    @Test
    public void testListUsers_Pagination() {
        given().spec(requestSpec)
            .queryParam("page", 1)
            .queryParam("size", 10)
        .when()
            .get("/users")
        .then()
            .statusCode(200)
            .body("content.size()", lessThanOrEqualTo(10))
            .body("totalPages", greaterThan(0))
            .body("currentPage", equalTo(1));
    }

    // ===== SECURITY =====
    @Test
    public void testGetUser_NoAuth_Returns401() {
        given()  // no auth header
            .contentType(ContentType.JSON)
        .when()
            .get("/users/1")
        .then()
            .statusCode(401);
    }

    @Test
    public void testDeleteUser_RegularUser_Returns403() {
        given().spec(requestSpec)
            .header("Authorization", "Bearer " + regularUserToken)
        .when()
            .delete("/users/" + adminUserId)
        .then()
            .statusCode(403);
    }
}
```

---

## 12. Behavioral / Situational Questions

### Q53. Tell me about yourself and your testing experience.

**Answer Template:**

> "I'm an Automation Test Engineer with [X] years of experience in designing and implementing test automation frameworks. I specialize in **Java** and **Selenium WebDriver** for UI automation, and **REST Assured** for API testing.
>
> In my current/previous role at [Company], I:
> - Built a **hybrid automation framework** from scratch using Page Object Model, reducing manual regression effort by 60%
> - Automated **200+ test cases** covering UI, API, and database validations
> - Integrated the framework with **Jenkins CI/CD pipeline**, enabling automated regression on every build
> - Reduced regression testing time from **3 days to 4 hours**
> - Collaborated closely with developers in **Agile/Scrum** environment, participating in sprint ceremonies and contributing to the definition of done
>
> I'm proficient in **SQL** for backend data validation and have experience with tools like TestNG, Maven, Git, and reporting tools like Extent Reports."

---

### Q54. How do you handle a situation where developers say "it's not a bug"?

**Answer:**

> 1. **Stay objective** — present facts, not opinions
> 2. **Show evidence** — screenshots, logs, API responses, steps to reproduce
> 3. **Reference requirements** — point to user stories, acceptance criteria, or specs
> 4. **Discuss impact** — explain how it affects end users
> 5. **Escalate if needed** — involve BA or Product Owner to clarify expected behavior
> 6. **Document everything** — even if it's classified as "by design," document the decision
>
> *"I've been in situations where a developer considered a behavior to be expected, but the acceptance criteria stated otherwise. I brought up the specific requirement in a sprint meeting, and we collectively decided it was indeed a defect. The key is to focus on the requirement, not on who is right."*

---

### Q55. Tell me about a time you improved the testing process.

**Answer Template (STAR Method):**

> **Situation:** "In my previous project, regression testing took 5 days manually, delaying releases."
>
> **Task:** "I was tasked with reducing regression time while maintaining quality."
>
> **Action:**
> - Analyzed existing manual test cases and identified 150 high-priority candidates for automation
> - Designed a Selenium + REST Assured framework with Page Object Model
> - Implemented data-driven testing to cover multiple scenarios per test
> - Set up parallel execution using Selenium Grid (4 threads)
> - Integrated with Jenkins for nightly runs
> - Created Extent Report dashboard for stakeholder visibility
>
> **Result:**
> - Regression time reduced from **5 days to 6 hours**
> - Test coverage increased by **40%**
> - Defect leakage to production reduced by **35%**
> - Team confidence in releases improved significantly

---

### Q56. How do you prioritize your testing when there's a tight deadline?

**Answer:**

> 1. **Risk-based testing** — Focus on high-risk, high-impact areas first
> 2. **Smoke test** — Ensure critical flows work (login, checkout, payment)
> 3. **Run automated regression** — Catch obvious regressions quickly
> 4. **Test new features** — Focus on the stories delivered in this sprint
> 5. **Communicate** — Inform the team about what will and won't be covered
> 6. **Document untested areas** — Flag risks in the test report
>
> *"I always maintain a risk matrix that categorizes features by business impact and change frequency. When time is tight, I test the high-risk, high-impact quadrant first and ensure automated regression covers the rest."*

---

### Q57. How do you stay current with testing tools and technologies?

**Answer:**

> - **Blogs & Newsletters:** Ministry of Testing, Google Testing Blog, Baeldung
> - **Communities:** Stack Overflow, LinkedIn testing groups, Reddit r/QualityAssurance
> - **Conferences:** SeleniumConf, TestBash, Google Test Automation Conference
> - **Hands-on:** Personal projects, contributing to open-source testing tools
> - **Certifications:** ISTQB Foundation, ISTQB Advanced Test Automation Engineer
> - **Podcasts:** Test & Code, AB Testing
> - **Tool Updates:** Follow Selenium, REST Assured, TestNG release notes

---

### Q58. Describe a challenging bug you found and how you debugged it.

**Answer Template (STAR Method):**

> **Situation:** "We had intermittent failures in the checkout flow — the order total was sometimes incorrect."
>
> **Task:** "I needed to find the root cause of this inconsistent behavior."
>
> **Action:**
> - Added detailed API logging to capture request/response at each step
> - Discovered the discount calculation API was returning different values for the same coupon
> - Used SQL queries to check the coupon table — found a race condition where two concurrent requests were modifying the same coupon's usage count
> - Wrote a **reproducible automated test** that hit the API concurrently with 10 threads using the same coupon
> - The test consistently showed the race condition
> - Filed a detailed bug report with API logs, DB queries, and the automated reproduction script
>
> **Result:**
> - Developer fixed it by adding a database-level lock on coupon usage
> - I added the concurrency test to the regression suite
> - No recurrence of the issue in production

---

## Quick Reference: Common Interview Code Challenges

### Challenge 1: Write a method to check if all links on a page are working

```java
public Map<String, Integer> checkBrokenLinks(WebDriver driver) {
    Map<String, Integer> results = new LinkedHashMap<>();
    List<WebElement> links = driver.findElements(By.tagName("a"));

    for (WebElement link : links) {
        String url = link.getAttribute("href");
        if (url == null || url.isEmpty() || url.startsWith("javascript")) continue;

        try {
            HttpURLConnection conn = (HttpURLConnection) new URL(url).openConnection();
            conn.setRequestMethod("HEAD");
            conn.setConnectTimeout(5000);
            conn.connect();
            results.put(url, conn.getResponseCode());
        } catch (Exception e) {
            results.put(url, -1); // unreachable
        }
    }

    // Filter broken links
    results.entrySet().stream()
        .filter(e -> e.getValue() >= 400 || e.getValue() == -1)
        .forEach(e -> System.out.println("BROKEN: " + e.getKey() + " → " + e.getValue()));

    return results;
}
```

### Challenge 2: Write a REST Assured test that validates pagination

```java
@Test
public void testPaginationAllPages() {
    int page = 1;
    int totalPages;
    List<String> allUserIds = new ArrayList<>();

    do {
        Response response = given().spec(requestSpec)
            .queryParam("page", page)
            .queryParam("size", 10)
        .when()
            .get("/users")
        .then()
            .statusCode(200)
            .extract().response();

        totalPages = response.path("totalPages");
        List<String> ids = response.jsonPath().getList("content.id");
        allUserIds.addAll(ids);
        page++;
    } while (page <= totalPages);

    // Verify no duplicates across pages
    Set<String> uniqueIds = new HashSet<>(allUserIds);
    assertEquals(uniqueIds.size(), allUserIds.size(), "Duplicate records found across pages!");
}
```

### Challenge 3: Write a utility to wait for an element with custom polling

```java
public static <T> T waitForCondition(WebDriver driver, Function<WebDriver, T> condition,
                                      int timeoutSec, int pollMs, String errorMessage) {
    FluentWait<WebDriver> wait = new FluentWait<>(driver)
        .withTimeout(Duration.ofSeconds(timeoutSec))
        .pollingEvery(Duration.ofMillis(pollMs))
        .ignoring(NoSuchElementException.class)
        .ignoring(StaleElementReferenceException.class)
        .withMessage(errorMessage);

    return wait.until(condition);
}

// Usage
WebElement element = waitForCondition(driver,
    d -> d.findElement(By.id("results")),
    30, 200, "Search results did not load");
```

---

> [!TIP]
> **Interview Day Tips:**
> - Review the company's product before the interview — prepare relevant examples
> - Be ready to write code on a whiteboard or shared screen
> - Explain your thought process as you solve problems
> - It's okay to say "I don't know" — then explain how you'd find the answer
> - Ask thoughtful questions about their test infrastructure, CI/CD, and team structure
