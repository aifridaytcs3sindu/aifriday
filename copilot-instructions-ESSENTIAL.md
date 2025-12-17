# REST Assured API Test Generator - Essential Instructions

**Purpose**: Generate complete REST Assured + Cucumber test suite from Swagger/OpenAPI specification.

**Usage**: `@workspace #file:copilot-instructions-ESSENTIAL.md Generate API tests from Swagger`

---

## 🎯 Core Workflow

1. **Parse Swagger** → Extract endpoints, schemas, authentication
2. **Parse Response Schemas** → Get actual field names (response/data/result)
3. **Generate Tests** → Feature files + testData.json + step definitions
4. **Execute** → Run tests, generate reports

---

## ⚠️ CRITICAL RULES

### 1. Swagger Response Schema Parsing

**MANDATORY**: Extract field names from Swagger - DON'T assume "data" or "response"

```
Swagger Navigation:
paths → /api/endpoint → post/get → responses → 200 
  → content → application/json → schema → $ref 
  → components → schemas → ResponseSchemaName → properties
```

**Example**:
```json
"ResponseMapStringListResponse": {
  "properties": {
    "response": { "type": "object" },  // ← Use this in assertions!
    "status": { "type": "boolean" },
    "error": { ... }
  }
}
```

**Feature File**:
```gherkin
And the response should contain "response"  # ✅ From schema
# NOT: And the response should contain "data"  # ❌ Assumption
```

### 2. Endpoint Path Alignment

- Feature file: `/api/dsbudget/v1/details` (actual API path)
- testData.json key: `dsbudget/v1/details` (strip `/api/` prefix)
- **NEVER invent endpoint paths** - use exact paths from Swagger

### 3. Data-Driven Approach

**testData.json structure**:
```json
{
  "endpoints": {
    "dsbudget/v1/details": {
      "request": {
        "franchiseNumber": [8439, 8345],
        "seasonName": "SM25",
        "businessUnit": ["G3-02"]
      },
      "expectedInResponse": {
        "status": true,
        "error": null
      }
    }
  }
}
```

**Step Definition** (5-15 lines max):
```java
@When("I send a POST request to {string} with valid payload")
public void i_send_post_request(String endpoint) {
    String key = endpoint.replace("/api/", "");
    Map<String, Object> payload = TestDataLoader.getRequestData(key);
    
    Response response = given()
        .spec(RequestSpecFactory.getAuthRequestSpec())
        .body(payload)
    .when()
        .post(endpoint);
    
    context.setResponse(response);
}
```

**❌ NEVER DO THIS**:
```java
// Hardcoded if-else blocks - 100+ lines!
if (endpoint.contains("windowsandcamp")) {
    payload.put("franchiseNumber", 9999);
    payload.put("season", "SM25");
} else if (endpoint.contains("virtualsalesplan")) {
    // ...
}
```

---

## 📋 Generation Checklist

### Parse Swagger Schema
- [ ] Extract response schema for each endpoint (200/201 responses)
- [ ] Navigate to `components.schemas.SchemaName.properties`
- [ ] Get actual field names (response/data/result/etc.)
- [ ] **Fallback**: Default to "response" if schema unavailable

### Generate Feature Files
```gherkin
@endpoint-name @positive
Scenario: Successfully call endpoint
  Given authentication is set up
  When I send a POST request to "/api/endpoint/v1/path" with valid payload
  Then the response status code should be 200
  And the response should have status "true"
  And the response should contain "response"  # ← From schema!

@endpoint-name @negative @no-auth
Scenario: Attempt without authentication
  When I send a POST request to "/api/endpoint/v1/path" without authentication
  Then the response status code should be 401
```

### Generate testData.json
- [ ] One entry per endpoint: `endpoints.<path>.request`
- [ ] Only business-critical fields (Swagger required fields auto-generated)
- [ ] Add `expectedInResponse` for validation

### Generate Step Definitions
- [ ] Use `TestDataLoader.getRequestData(endpoint)`
- [ ] 5-15 lines per step
- [ ] No if-else blocks for payloads

### Generate Helpers
- [ ] `TestDataLoader.java` - Load from testData.json
- [ ] `RequestSpecFactory.java` - Auth request specs
- [ ] `ConfigReader.java` - Read .env file

---

## 🔧 Technical Stack

**Dependencies**:
- Java 17
- Maven 3.8+
- REST Assured 5.3+
- Cucumber 7.14+
- TestNG 7.8+
- Jackson Databind 2.15+

**pom.xml**:
```xml
<dependencies>
    <!-- REST Assured -->
    <dependency>
        <groupId>io.rest-assured</groupId>
        <artifactId>rest-assured</artifactId>
        <version>5.3.2</version>
    </dependency>
    <dependency>
        <groupId>io.rest-assured</groupId>
        <artifactId>json-schema-validator</artifactId>
        <version>5.3.2</version>
    </dependency>
    
    <!-- Cucumber -->
    <dependency>
        <groupId>io.cucumber</groupId>
        <artifactId>cucumber-java</artifactId>
        <version>7.14.0</version>
    </dependency>
    <dependency>
        <groupId>io.cucumber</groupId>
        <artifactId>cucumber-testng</artifactId>
        <version>7.14.0</version>
    </dependency>
    
    <!-- TestNG -->
    <dependency>
        <groupId>org.testng</groupId>
        <artifactId>testng</artifactId>
        <version>7.8.0</version>
    </dependency>
    
    <!-- JSON Processing -->
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.15.3</version>
    </dependency>
    
    <!-- Extent Reports -->
    <dependency>
        <groupId>com.aventstack</groupId>
        <artifactId>extentreports</artifactId>
        <version>5.1.1</version>
    </dependency>
    
    <!-- Allure Reports -->
    <dependency>
        <groupId>io.qameta.allure</groupId>
        <artifactId>allure-cucumber7-jvm</artifactId>
        <version>2.24.0</version>
    </dependency>
    <dependency>
        <groupId>io.qameta.allure</groupId>
        <artifactId>allure-rest-assured</artifactId>
        <version>2.24.0</version>
    </dependency>
    
    <!-- Logging -->
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>2.0.9</version>
    </dependency>
    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-classic</artifactId>
        <version>1.4.11</version>
    </dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>3.1.2</version>
            <configuration>
                <suiteXmlFiles>
                    <suiteXmlFile>testng.xml</suiteXmlFile>
                </suiteXmlFiles>
            </configuration>
        </plugin>
        <plugin>
            <groupId>io.qameta.allure</groupId>
            <artifactId>allure-maven</artifactId>
            <version>2.12.0</version>
        </plugin>
    </plugins>
</build>
```

---

## 🔐 Authentication

**.env file**:
```properties
BASE_URL=https://api.example.com/service
AUTH_TYPE=jwt  # Options: basic, bearer, jwt, none
# JWT
JWT_TOKEN=<token>
JWT_USER_ID=850680
JWT_APP_ID=rb
JWT_USER_TEAM=drb
# Basic Auth
AUTH_USERNAME=admin
AUTH_PASSWORD=secret
# Bearer Token
AUTH_TOKEN=<bearer-token>
```

**RequestSpecFactory.java**:
```java
public static RequestSpecification getAuthRequestSpec() {
    String authType = ConfigReader.getProperty("AUTH_TYPE");
    RequestSpecification spec = new RequestSpecBuilder()
        .setBaseUri(ConfigReader.getProperty("BASE_URL"))
        .setContentType(ContentType.JSON)
        .build();
    
    switch(authType) {
        case "basic":
            return spec.auth().preemptive().basic(
                ConfigReader.getProperty("AUTH_USERNAME"),
                ConfigReader.getProperty("AUTH_PASSWORD"));
        case "bearer":
            return spec.header("Authorization", 
                "Bearer " + ConfigReader.getProperty("AUTH_TOKEN"));
        case "jwt":
            return spec
                .header("X-Auth-Token", ConfigReader.getProperty("JWT_TOKEN"))
                .header("X-MS-IBT-User-ID", ConfigReader.getProperty("JWT_USER_ID"))
                .header("X-MS-APP-ID", ConfigReader.getProperty("JWT_APP_ID"))
                .header("X-User-Team", ConfigReader.getProperty("JWT_USER_TEAM"));
        default:
            return spec;  // No auth
    }
}
```

---

## 📁 Project Structure

```
src/
├── test/
│   ├── java/com/api/automation/
│   │   ├── runners/TestRunner.java
│   │   ├── stepdefinitions/
│   │   │   ├── CommonSteps.java
│   │   │   ├── PostSteps.java
│   │   │   └── GetSteps.java
│   │   ├── hooks/Hooks.java
│   │   └── config/
│   │       ├── ConfigReader.java
│   │       └── TestContext.java
│   ├── helpers/
│   │   ├── TestDataLoader.java
│   │   └── RequestSpecFactory.java
│   └── resources/
│       ├── features/*.feature
│       ├── testdata/testData.json
│       └── schemas/*.json
├── pom.xml
├── testng.xml
└── .env
```

---

## 🚨 Common Mistakes to Avoid

| ❌ NEVER | ✅ ALWAYS |
|---------|---------|
| Assume field is "data" | Parse Swagger schema for actual field name |
| Invent endpoint paths | Use exact paths from Swagger |
| Hardcode payloads in steps | Use `TestDataLoader.getRequestData()` |
| 100+ line step definitions | Keep steps 5-15 lines max |
| Auto-generate optional fields | Only generate REQUIRED fields |

---

## 🐛 Troubleshooting

| Error | Solution |
|-------|----------|
| `JSON path data doesn't match` | Parse Swagger: `components.schemas.SchemaName.properties` → use actual field name |
| `404 Not Found` | Verify endpoint in feature file matches Swagger path exactly |
| Wrong field name | Check schema `properties` - use "response"/"data"/"result" from schema |

---

## 📊 Reporting Configuration

### Cucumber Reports (Auto-generated)
**TestRunner.java**:
```java
@CucumberOptions(
    features = "src/test/resources/features",
    glue = {"com.api.automation.stepdefinitions", "com.api.automation.hooks"},
    plugin = {
        "pretty",
        "html:reports/cucumber-reports/cucumber.html",
        "json:reports/cucumber-reports/cucumber.json",
        "io.qameta.allure.cucumber7jvm.AllureCucumber7Jvm"
    },
    monochrome = true
)
```

### Extent Reports
**extent.properties**:
```properties
extent.reporter.spark.start=true
extent.reporter.spark.out=reports/extent-reports/ExtentReport.html
```

**Hooks.java**:
```java
import com.aventstack.extentreports.ExtentReports;
import com.aventstack.extentreports.ExtentTest;
import com.aventstack.extentreports.reporter.ExtentSparkReporter;

public class Hooks {
    private static ExtentReports extent;
    private static ThreadLocal<ExtentTest> test = new ThreadLocal<>();
    
    @BeforeAll
    public static void setupExtent() {
        ExtentSparkReporter spark = new ExtentSparkReporter(
            "reports/extent-reports/ExtentReport.html");
        extent = new ExtentReports();
        extent.attachReporter(spark);
    }
    
    @Before
    public void startTest(Scenario scenario) {
        ExtentTest extentTest = extent.createTest(scenario.getName());
        test.set(extentTest);
    }
    
    @After
    public void endTest(Scenario scenario) {
        if (scenario.isFailed()) {
            test.get().fail("Scenario failed");
        } else {
            test.get().pass("Scenario passed");
        }
    }
    
    @AfterAll
    public static void flushExtent() {
        extent.flush();
    }
}
```

### Allure Reports
**Add to RequestSpecFactory**:
```java
import io.qameta.allure.restassured.AllureRestAssured;

.addFilter(new AllureRestAssured())  // Add this to spec
```

**Generate Reports**:
```bash
mvn allure:serve      # Opens interactive report
mvn allure:report     # Generates static HTML
```

---

## 🎬 Quick Start

1. **Read** `.env` for BASE_URL, AUTH_TYPE, credentials
2. **Parse** Swagger response schemas → extract field names
3. **Generate**:
   - testData.json (endpoints structure)
   - Feature files (use schema field names!)
   - Step definitions (5-15 lines, use TestDataLoader)
   - Helpers (RequestSpecFactory, TestDataLoader, ConfigReader)
   - Hooks (with Extent Reports integration)
4. **Run**: `mvn clean test`
5. **Reports**: 
   - Cucumber: `reports/cucumber-reports/cucumber.html`
   - Extent: `reports/extent-reports/ExtentReport.html`
   - Allure: `mvn allure:serve`

---

**This API Pattern**: All responses use `"response"` field (verified from Swagger schemas)

**Key Principle**: Parse Swagger schemas → Don't assume → Let tests fail if API is wrong
