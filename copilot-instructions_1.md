# API Test Generator - REST Assured + Cucumber

**Description**: Automatically generates complete REST Assured + Cucumber API test suites from Swagger/OpenAPI specifications with JSON schema validation, flexible authentication (Basic, Bearer, JWT), and comprehensive reporting.

**Usage**: `Generate API tests` or `@workspace #file:copilot-instructions.md Generate API tests`

---

## Agent Instructions

Generate a **complete REST Assured + Cucumber API test suite in Java 17** for all endpoints defined in the API specification configured in the `.env` file, with:
- JSON Schema validation using REST Assured's schema validation
- Flexible authentication (basic/bearer/jwt/none)
- Multiple report formats (Cucumber HTML/JSON, Extent Reports, TEST_SUMMARY.md)
- Automatic retry logic (3 attempts per scenario)
- **Data-driven testing** using centralized `testData.json` (no hardcoded payloads in step definitions)
- Swagger schema parsing with auto-generation utilities (SwaggerParser, DataGenerator)
- RequestSpecification and ResponseSpecification for reusability

### Framework Architecture Principles

**CRITICAL - Read This First When Setting Up New Project**:

1. **Data-Driven Approach (MANDATORY)**:
   - Define only business-critical field values in `src/test/resources/testdata/testData.json`
   - Step definitions should NEVER contain hardcoded if-else blocks for building payloads
   - Use `TestDataLoader.getRequestData(endpoint)` method to dynamically build complete payloads
   - **AUTO-GENERATION**: Required fields missing from testData.json are auto-generated from Swagger schema
   - This keeps code DRY, maintainable, and scalable

2. **Centralized Test Data Structure** (`testData.json`):
   - `common` section: Reference data (NOT auto-merged into requests)
   - `endpoints.<path>.request`: Endpoint-specific request fields
   - `endpoints.<path>.expectedInResponse`: Expected values for validation

3. **Smart Auto-Generation**: Only REQUIRED fields (per Swagger schema) are auto-generated if missing from testData.json. Manual data always takes precedence. Optional fields are NOT auto-generated to avoid unwanted API data.

4. **Three-Level Validation**: Schema structure → Data types → Key business values

5. **Clean Step Definitions**:
   - Each step definition: **5-15 lines max**
   - No business logic - all data resolution in `TestDataLoader.java`
   - All API calls in helpers (`BaseAPIHelper.java`)
   - Use `TestDataLoader.getRequestData(endpoint)` for all POST/PUT/PATCH requests

**Example of CORRECT Implementation** (PostSteps.java):
```java
@When("I send a POST request to {string} with valid payload")
public void i_send_post_request_with_valid_payload(String endpoint) {
    // Strip /api/ prefix to match testData.json keys
    // IMPORTANT: endpoint in feature file MUST match the actual API path
    // Example: Feature uses "/api/dsbudget/v1/details" → testData.json key is "dsbudget/v1/details"
    String endpointPath = endpoint.replace("/api/", "");
    Map<String, Object> payload = TestDataLoader.getRequestData(endpointPath);
    context.setRequestBody(payload);
    
    Response response = given()
        .spec(RequestSpecFactory.getAuthRequestSpec())
        .body(payload)
    .when()
        .post(endpoint);
    
    context.setResponse(response);
}

@Then("the response status code should be {int}")
public void the_response_status_code_should_be(Integer statusCode) {
    context.getResponse().then()
        .statusCode(statusCode);  // Use REST Assured's fluent assertion
}
```

**Example of WRONG Implementation** (DON'T DO THIS):
```java
// ❌ ANTI-PATTERN - Hardcoded if-else blocks
@When("I send a POST request to {string} with valid payload")
public void i_send_post_request_with_valid_payload(String endpoint) {
    Map<String, Object> payload = new HashMap<>();
    
    if (endpoint.contains("windowsandcamp")) {
        payload.put("franchiseNumber", 9999);
        payload.put("season", "SM25");
        payload.put("type", "PH");
    } else if (endpoint.contains("virtualsalesplan")) {
        payload.put("franchiseNumber", 9999);
        payload.put("season", "SM25");
    } else if (endpoint.contains("vspbasket")) {
        payload.put("moduleName", "Basket");
        payload.put("sortBy", null);
        payload.put("sortOrder", "desc");
    }
    // ... 100+ more lines of if-else blocks
}
```

---

## Swagger Response Schema Parsing Guide

**CRITICAL for Feature File Generation**: Always parse Swagger/OpenAPI response schemas to extract actual field names instead of assuming `"data"` or `"response"`.

### Step-by-Step Schema Parsing Process

1. **Locate Endpoint Response Definition**:
   ```json
   // In swagger.json: paths → /api/endpoint → post/get → responses
   "/api/virtualsalesplan/v1/listing": {
     "post": {
       "responses": {
         "200": {
           "description": "OK",
           "content": {
             "application/json": {
               "schema": {
                 "$ref": "#/components/schemas/ResponseMapStringListSalesPlanListingResponse"
               }
             }
           }
         }
       }
     }
   }
   ```

2. **Resolve Schema Reference**:
   ```json
   // Navigate to: components → schemas → [SchemaName]
   "components": {
     "schemas": {
       "ResponseMapStringListSalesPlanListingResponse": {
         "type": "object",
         "properties": {
           "response": {              // ← THIS is the actual field name!
             "type": "object",
             "additionalProperties": {...}
           },
           "status": {
             "type": "boolean"
           },
           "error": {
             "$ref": "#/components/schemas/ErrorResponse"
           },
           "totalRecords": { "type": "integer" },
           // ... other fields
         }
       }
     }
   }
   ```

3. **Extract Top-Level Properties**:
   - From `properties` object, identify main data field: `"response"`, `"data"`, `"result"`, etc.
   - Note common fields: `"status"` (boolean), `"error"` (object/null)

4. **Generate Correct Assertions**:
   ```gherkin
   # Based on schema above, use:
   And the response should contain "response"    # ✅ Correct - from schema
   And the response should have status "true"    # ✅ Validates status field
   
   # DON'T assume:
   And the response should contain "data"        # ❌ Wrong - not in this schema
   ```

### Real-World Examples from This Project

| Endpoint | Schema Name | Main Data Field | Correct Assertion |
|----------|-------------|-----------------|-------------------|
| `/api/virtualsalesplan/v1/listing` | `ResponseMapStringListSalesPlanListingResponse` | `response` | `And the response should contain "response"` |
| `/api/windowsandcamp/v1/listing` | `ResponseMapStringListWindowsCampaignsListingResponse` | `response` | `And the response should contain "response"` |
| `/api/dsbudget/v1/details` | `ResponseListBudgetResponse` | `response` | `And the response should contain "response"` |
| `/api/favourites/v1/listing` | `ResponseListFavouritesResponse` | `response` | `And the response should contain "response"` |

### Common Patterns in This API

All response schemas follow this structure:
```json
{
  "response": { /* actual data - object, array, or map */ },
  "status": true,
  "error": null,
  "totalRecords": null,
  "filterValues": null,
  // ... additional metadata fields
}
```

**Key Takeaway**: This API consistently uses `"response"` (not `"data"`) as the main data field. Always verify via schema parsing, not assumptions.

### Configuration Source


Read all configuration from `.env` file in project root:
- `BASE_URL`: API base URL endpoint
- `AUTH_TYPE`: Authentication method (`basic`, `bearer`, `jwt`, `none`)
- `AUTH_USERNAME` / `AUTH_PASSWORD`: For Basic Auth (if using basic auth)
- `AUTH_TOKEN`: For Bearer Token Auth (if using bearer auth)
- `JWT_TOKEN`, `JWT_USER_ID`, `JWT_APP_ID`, `JWT_USER_TEAM`: For JWT Auth (required for jwt auth)
- `REQUEST_TIMEOUT`: API request timeout in milliseconds (default: 30000)

**Note**: Sensitive values (tokens, passwords) should be set as system environment variables in CI/CD and NOT committed to Git.

### Technical Stack

- **Java 17+** (LTS version)
- **Maven 3.8+** (build tool)
- **REST Assured 5.3+** (API testing library)
- **Cucumber 7.14+** (BDD framework)
- **TestNG 7.8+** (test execution framework)
- **Jackson Databind 2.15+** (JSON serialization/deserialization)
- **JSON Schema Validator** (for schema validation)
- **Extent Reports 5.1+** (HTML reporting)
- **Allure 2.24+** (test reporting and dashboard)
- **SLF4J + Logback** (logging framework)
- **Apache Commons Configuration** (configuration management)

---

## Important Implementation Details

### SSL Certificate Handling
Always configure REST Assured to accept all SSL certificates for corporate environments:
```java
RestAssured.useRelaxedHTTPSValidation();
```

### Base URI and Path Construction
- Configure `baseURI` in RequestSpecification
- Use `basePath` for API version prefix (e.g., `/api/v1`)
- Endpoint paths should NOT include leading `/` to prevent conflicts
- Example: `.basePath("/api")` + endpoint `"users"` = `/api/users`

### Content Type Management
Always set and validate content types:
```java
// JSON (most common)
.contentType(ContentType.JSON)
.accept(ContentType.JSON)

// XML
.contentType(ContentType.XML)
.accept(ContentType.XML)

// Form data
.contentType(ContentType.URLENC)

// Multiple accept types
.accept(ContentType.JSON, ContentType.XML)
```

### Global Configuration vs Request-Level Configuration

**Option 1: Global Configuration (in Hooks or Base Test)**
```java
@BeforeClass
public static void setup() {
    RestAssured.baseURI = "https://api.example.com";
    RestAssured.basePath = "/api/v1";
    RestAssured.port = 443;
    RestAssured.useRelaxedHTTPSValidation();
    
    // Global request configuration
    RestAssured.requestSpecification = new RequestSpecBuilder()
        .setContentType(ContentType.JSON)
        .addFilter(new RequestLoggingFilter())
        .addFilter(new ResponseLoggingFilter())
        .build();
    
    // Global response configuration
    RestAssured.responseSpecification = new ResponseSpecBuilder()
        .expectContentType(ContentType.JSON)
        .build();
}

@AfterClass
public static void cleanup() {
    RestAssured.reset();  // Reset all configurations
}
```

**Option 2: Request-Level Specification (Recommended for flexibility)**
```java
// Use RequestSpecFactory and ResponseSpecFactory as shown in section 1 & 2
given()
    .spec(RequestSpecFactory.getAuthRequestSpec())
.when()
    .get("/users")
.then()
    .spec(ResponseSpecFactory.getSuccessResponseSpec());
```

**Best Practice**: Use RequestSpecification factories for better control and testability, avoiding global state.

---

## Project Structure

```
src/
├── main/
│   └── java/
│       └── com/api/automation/
│           ├── config/
│           │   ├── ConfigReader.java              # Properties file reader
│           │   └── TestContext.java               # Shared test context
│           ├── endpoints/
│           │   └── APIEndpoints.java              # Endpoint constants
│           ├── helpers/
│           │   ├── BaseAPIHelper.java             # Base REST Assured helper
│           │   ├── TestDataLoader.java            # Test data loader with validation
│           │   ├── RetryHelper.java               # Retry logic utilities
│           │   └── SchemaValidator.java           # JSON Schema validation
│           ├── models/
│           │   ├── request/
│           │   │   └── <Endpoint>Request.java     # Request POJOs
│           │   └── response/
│           │       └── <Endpoint>Response.java    # Response POJOs
│           ├── specifications/
│           │   ├── RequestSpecFactory.java        # RequestSpecification factory
│           │   └── ResponseSpecFactory.java       # ResponseSpecification factory
│           └── utils/
│               ├── LoggerUtil.java                # Custom logger
│               └── DateTimeUtil.java              # Date/time utilities
├── test/
│   ├── java/
│   │   └── com/api/automation/
│   │       ├── runners/
│   │       │   └── TestRunner.java                # Cucumber TestNG runner
│   │       ├── stepdefinitions/
│   │       │   ├── CommonSteps.java               # Common step definitions
│   │       │   ├── PostSteps.java                 # POST request steps
│   │       │   ├── GetSteps.java                  # GET request steps
│   │       │   └── <Endpoint>Steps.java           # Endpoint-specific steps
│   │       └── hooks/
│   │           └── Hooks.java                     # Before/After hooks
│   └── resources/
│       ├── features/
│       │   ├── <endpoint1>.feature                # Gherkin scenarios
│       │   └── <endpoint2>.feature
│       ├── schemas/
│       │   ├── <endpoint1>-response-schema.json   # JSON schemas
│       │   └── <endpoint2>-response-schema.json
│       ├── testdata/
│       │   └── testData.json                      # Centralized test data
│       ├── logback.xml                            # Logging configuration
│       └── extent.properties                      # Extent report configuration
reports/                                            # Generated test reports
├── cucumber-reports/
│   ├── cucumber.html
│   └── cucumber.json
├── extent-reports/
│   └── ExtentReport.html
└── allure-results/                                # Allure test results
allure-report/                                     # Allure HTML report (generated)
pom.xml                                            # Maven dependencies
testng.xml                                         # TestNG suite configuration
TEST_SUMMARY.md                                    # Test execution summary (generated)
.env.example                                       # Environment template
```

### Key Helper Functions

**TestDataLoader.java** provides the following methods:

| Method | Purpose | Returns |
|--------|---------|---------|
| `getRequestData(String endpoint)` | Get request payload for an endpoint | `Map<String, Object>` with request fields |
| `getExpectedInResponse(String endpoint)` | Get expected values for response validation | `Map<String, Object>` with expected field values |
| `validateExpectedValues(Response response, String endpoint)` | Validate response against expected values | `ValidationResult` object with matches/mismatches |
| `getEndpointData(String endpoint)` | Get merged common + request data | `Map<String, Object>` with all fields |
| `getCommonData()` | Get shared common data | `Map<String, Object>` with common fields |

**Naming Convention**: 
- Classes: PascalCase (`OrderSteps.java`, `StoreAPIHelper.java`)
- Methods: camelCase (`createOrder()`, `getOrderById()`)
- Packages: lowercase (`com.api.automation.stepdefinitions`)
- Constants: UPPER_SNAKE_CASE (`CREATE_ORDER`, `BASE_URL`)

---

## Test Data Management

### Centralized Test Data

**Location**: `src/test/resources/testdata/testData.json`

**Purpose**: Provides reusable test data values organized by endpoint, enabling GitHub Copilot to reference specific data for each API endpoint without ambiguity. Also defines expected response values for key field validation.

**Structure**: Hierarchical structure with common data, endpoint-specific request data, and expected response values

```json
{
  "description": "Test data organized by endpoint with request data and expected response values for validation",
  
  "common": {
    "userId": "850680",
    "userType": "global",
    "franchiseeList": [8439, 8345, 5384]
  },

  "endpoints": {
    "windowsandcamp/v1/listing": {
      "request": {
        "franchiseNumber": 9999,
        "season": "SM25",
        "type": "PH"
      },
      "expectedInResponse": {
        "status": true,
        "error": null
      },
      "_comment": "Response contains sales plans grouped by userId"
    },
    "optionsplanning/budget/overallbudget/listing": {
      "request": {
        "account": 9328,
        "season": "SM25",
        "businessUnit": "WOMENSWEAR"
      },
      "expectedInResponse": {
        "status": true,
        "response.account": 9328,
        "response.season": "SM25"
      },
      "_comment": "Validates that response echoes the requested account and season"
    }
  }
}
```

### Structure Explained

| Section | Purpose | Example |
|---------|---------|---------|
| `common` | Shared data across all endpoints | userId, franchiseeList |
| `endpoints.<path>.request` | Request payload data for the endpoint | season, account, businessUnit |
| `endpoints.<path>.expectedInResponse` | Key values to validate in response | status, message, echoed fields |
| `endpoints.<path>._comment` | Documentation for the endpoint | Describes what response contains |

### Three-Level Validation Approach

The test framework performs **3 levels of validation** for comprehensive API testing:

| Level | What it Validates | How | When |
|-------|------------------|-----|------|
| **1. Schema Validation** | Response structure matches expected JSON schema | REST Assured's `matchesJsonSchema()` against schemas in `schemas/` folder | Always for 200/201 responses |
| **2. Data Type Validation** | Fields have correct types (string, number, boolean, array, object) | Part of schema validation - `type` property in JSON schema | Always for 200/201 responses |
| **3. Key Values Validation** | Specific business-critical values match expected | `expectedInResponse` in `testData.json` validated by `validateExpectedValues()` | For positive scenarios |

#### Key Values Validation Examples

```json
{
  "expectedInResponse": {
    "status": true,                    // Top-level field
    "error": null,                     // Null check
    "response.account": 9328,          // Nested field (dot notation)
    "response.season": "SM25",         // String match
    "message": "Saved/Updated Successfully"  // Success message
  }
}
```

**Dot Notation**: Use `response.fieldName` to validate nested fields in the response object using JsonPath.

### Usage Guidelines

**Import**: 
```java
import com.api.automation.helpers.TestDataLoader;
import java.util.Map;

Map<String, Object> requestData = TestDataLoader.getRequestData("endpoint/path");
Map<String, Object> expectedData = TestDataLoader.getExpectedInResponse("endpoint/path");
```

---

## Test Coverage Requirements

### Positive Scenarios (200/201/204)
- Validate successful operations per Swagger/OpenAPI definitions
- Apply three-level validation (see "Three-Level Validation Approach" section above)
- Use status codes defined in specification
- Test failures indicate API bugs

### Feature File Pattern

**CRITICAL: Endpoint Path Alignment**
- Feature file endpoints MUST exactly match the actual API paths (what you send to the server)
- testData.json keys MUST match the endpoint path with `/api/` prefix removed
- **Example**:
  - Actual API endpoint: `POST https://example.com/api/dsbudget/v1/details`
  - Feature file: `When I send a POST request to "/api/dsbudget/v1/details" with valid payload`
  - testData.json key: `"dsbudget/v1/details": { "request": {...} }`
- **DO NOT invent or guess endpoint paths** - use exact paths from API documentation or testData.json

```gherkin
@positive
Scenario: Successfully get overall budget listing
  Given authentication is set up
  When I send a POST request to "/api/optionsplanning/budget/overallbudget/listing" with valid payload
  Then the response status code should be 200
  And the response should have status "true"
  And the response should contain "response"  # Most APIs return data in "response" field
```

**CRITICAL: Response Structure Validation**
- **Extract field names from Swagger response schema** (see "Parse API Specification" workflow)
- Most APIs in this project return data in a `"response"` field (NOT `"data"`)
- **DO NOT assume field names** - parse them from Swagger `components.schemas` → response schema → `properties`
- Common patterns in this API (from Swagger schemas):
  - Success responses: `{ "response": {...}, "status": true, "error": null }`
  - Some endpoints may use `"data"` instead - check the actual schema!
- Use step: `And the response should contain "response"` to verify the field exists and is not null
- To validate specific nested values, add them to `expectedInResponse` in testData.json
- **Verification steps**:
  1. Look up endpoint in Swagger: `paths → /api/endpoint → post/get → responses → 200`
  2. Find schema reference: `content → application/json → schema → $ref`
  3. Resolve reference: `components → schemas → ResponseSchemaName → properties`
  4. Use actual property names from schema in assertions

**Note on Response Time Assertions**:
- Response time assertions have been **removed** from feature files
- Functional API tests should focus on correctness, not performance
- Use dedicated performance testing tools (JMeter, Gatling, k6) for performance validation
- If you need to add response time checks back, use: `And the response time should be less than {int} milliseconds`

### Negative Scenarios

**Always generate** 401 Unauthorized tests for all HTTP methods (GET, POST, PUT, DELETE) to ensure the API implements proper authentication, even if not documented in Swagger.

For other status codes (400, 403, 404, 500), only generate test cases if explicitly documented in the Swagger/OpenAPI specification.

---

## Authentication Handling

| AUTH_TYPE | Variables | Implementation |
|-----------|-----------|----------------|
| `basic` | `AUTH_USERNAME`, `AUTH_PASSWORD` | `.auth().preemptive().basic(username, password)` |
| `bearer` | `AUTH_TOKEN` | `.header("Authorization", "Bearer " + token)` |
| `jwt` | `JWT_TOKEN`, `JWT_USER_ID`, `JWT_APP_ID`, `JWT_USER_TEAM` | Multiple headers: `X-Auth-Token`, `X-MS-IBT-User-ID`, `X-MS-APP-ID`, `X-User-Team` |
| `none` | - | No auth headers |

**401 Testing**: For negative auth tests, use a separate spec without authentication headers or with invalid credentials.

---

## REST Assured Best Practices

### Required Static Imports

**CRITICAL**: Add these static imports at the top of your test classes for idiomatic REST Assured syntax:

```java
import static io.restassured.RestAssured.*;
import static io.restassured.matcher.RestAssuredMatchers.*;
import static org.hamcrest.Matchers.*;
import static io.restassured.module.jsv.JsonSchemaValidator.matchesJsonSchemaInClasspath;
```

### 1. RequestSpecification Factory Pattern

Create reusable request specifications:

```java
public class RequestSpecFactory {
    public static RequestSpecification getBaseRequestSpec() {
        return new RequestSpecBuilder()
            .setBaseUri(ConfigReader.getProperty("BASE_URL"))
            .setContentType(ContentType.JSON)
            .addFilter(new RequestLoggingFilter())
            .addFilter(new ResponseLoggingFilter())
            .build();
    }
    
    public static RequestSpecification getAuthRequestSpec() {
        String authType = ConfigReader.getProperty("AUTH_TYPE");
        RequestSpecification spec = getBaseRequestSpec();
        
        switch(authType) {
            case "basic":
                return spec.auth().preemptive().basic(
                    ConfigReader.getProperty("AUTH_USERNAME"),
                    ConfigReader.getProperty("AUTH_PASSWORD")
                );
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
                return spec;
        }
    }
}
```

### 2. ResponseSpecification for Common Validations

```java
public class ResponseSpecFactory {
    public static ResponseSpecification getSuccessResponseSpec() {
        return new ResponseSpecBuilder()
            .expectStatusCode(200)
            .expectContentType(ContentType.JSON)
            .expectResponseTime(lessThan(5000L))
            .build();
    }
    
    public static ResponseSpecification getCreatedResponseSpec() {
        return new ResponseSpecBuilder()
            .expectStatusCode(201)
            .expectContentType(ContentType.JSON)
            .build();
    }
}
```

### 3. JSON Schema Validation

```java
import static io.restassured.module.jsv.JsonSchemaValidator.matchesJsonSchemaInClasspath;

Response response = given()
    .spec(RequestSpecFactory.getAuthRequestSpec())
    .body(requestBody)
.when()
    .post(APIEndpoints.CREATE_ORDER)
.then()
    .body(matchesJsonSchemaInClasspath("schemas/order-response-schema.json"))
    .extract().response();
```

### 4. POJO Serialization/Deserialization

**Request:**
```java
OrderRequest orderRequest = new OrderRequest();
orderRequest.setProductId(1);
orderRequest.setQuantity(5);

Response response = given()
    .spec(RequestSpecFactory.getAuthRequestSpec())
    .body(orderRequest)  // Automatic serialization
.when()
    .post(APIEndpoints.CREATE_ORDER);
```

**Response:**
```java
OrderResponse orderResponse = response.as(OrderResponse.class);  // Automatic deserialization
assertThat(orderResponse.getStatus()).isEqualTo("success");
```

### 5. JsonPath for Response Extraction

```java
Response response = given().get(APIEndpoints.GET_ORDER + "/123");

// Extract single value
String status = response.jsonPath().getString("status");
int orderId = response.jsonPath().getInt("response.orderId");

// Extract list
List<String> productNames = response.jsonPath().getList("products.name");

// Extract with filter
List<Map<String, Object>> activeOrders = response.jsonPath()
    .getList("orders.findAll { it.status == 'active' }");
```

### 6. Hamcrest Matchers for Assertions

**REST Assured Best Practice**: Use Hamcrest matchers in `.then()` for fluent, readable assertions:

```java
// Single field validation
given()
    .get("/api/users/1")
.then()
    .body("name", equalTo("John Doe"))
    .body("age", greaterThan(18))
    .body("email", containsString("@"))
    .body("active", is(true));

// Multiple assertions
given()
    .get("/api/orders")
.then()
    .body("orders.size()", greaterThan(0))
    .body("orders[0].id", notNullValue())
    .body("orders.status", everyItem(equalTo("active")))
    .body("orders.total", hasItems(100.0, 200.0));

// Nested field validation
given()
    .get("/api/product/123")
.then()
    .body("product.price", greaterThanOrEqualTo(0.0))
    .body("product.inventory.quantity", lessThan(1000))
    .body("product.tags", hasSize(3));

// Common Hamcrest Matchers:
// equalTo(), is(), not(), notNullValue(), nullValue()
// greaterThan(), lessThan(), greaterThanOrEqualTo(), lessThanOrEqualTo()
// containsString(), startsWith(), endsWith()
// hasItems(), hasItem(), hasSize(), everyItem()
```

### 7. Request/Response Logging

```java
// Method 1: Using Filters (recommended for all requests)
given()
    .filter(new RequestLoggingFilter())
    .filter(new ResponseLoggingFilter())
    .get("/endpoint");

// Method 2: Using log() method
given()
    .log().all()  // Log request
.when()
    .get("/endpoint")
.then()
    .log().all();  // Log response

// Method 3: Conditional logging (only on failure)
given()
    .log().ifValidationFails()
.when()
    .get("/endpoint")
.then()
    .log().ifError();

// Method 4: Log specific parts
given()
    .log().headers()  // Only headers
    .log().body()     // Only body
.when()
    .get("/endpoint")
.then()
    .log().status()   // Only status code
    .log().body();    // Only response body
```

### 8. Query Parameters and Path Parameters

```java
// Query parameters
given()
    .queryParam("season", "SM25")
    .queryParam("type", "PH")
    .get("/listing");

// Path parameters
given()
    .pathParam("orderId", 12345)
    .get("/orders/{orderId}");

// Multiple parameters
Map<String, Object> params = new HashMap<>();
params.put("season", "SM25");
params.put("account", 9328);

given()
    .queryParams(params)
    .get("/budget");
```

### 9. Headers Management

```java
// Single header
given()
    .header("Accept-Language", "en-US")
    .get("/endpoint");

// Multiple headers
Map<String, String> headers = new HashMap<>();
headers.put("Accept-Language", "en-US");
headers.put("User-Agent", "APITestFramework/1.0");

given()
    .headers(headers)
    .get("/endpoint");
```

---

## Configuration Management

### .env File Structure

```properties
# API Configuration
BASE_URL=https://mns-prod-test.apigee.net/dsOrchService

# Authentication (JWT)
AUTH_TYPE=jwt
JWT_USER_ID=850680
JWT_APP_ID=rb
JWT_USER_TEAM=drb
# JWT_TOKEN should be set as system environment variable (not in .env file for security)

# Request Configuration (optional)
REQUEST_TIMEOUT=30000
```

### ConfigReader Implementation

```java
public class ConfigReader {
    private static final Dotenv dotenv;
    
    static {
        // Load .env file from project root
        dotenv = Dotenv.configure()
                .directory(".")
                .ignoreIfMissing()
                .load();
    }
    
    public static String getProperty(String key) {
        // Priority 1: System environment variables
        String value = System.getenv(key);
        if (value != null) {
            return value;
        }
        
        // Priority 2: .env file
        return dotenv.get(key);
    }
    
    public static String getProperty(String key, String defaultValue) {
        String value = getProperty(key);
        return value != null ? value : defaultValue;
    }
}
```

---

## Retry Logic Implementation

### Cucumber Retry Hook

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
public class TestRunner extends AbstractTestNGCucumberTests {
    
    @Override
    @DataProvider(parallel = false)
    public Object[][] scenarios() {
        return super.scenarios();
    }
}
```

### TestNG Retry Analyzer

```java
public class RetryAnalyzer implements IRetryAnalyzer {
    private int retryCount = 0;
    private static final int MAX_RETRY_COUNT = 3;
    
    @Override
    public boolean retry(ITestResult result) {
        if (retryCount < MAX_RETRY_COUNT) {
            retryCount++;
            return true;
        }
        return false;
    }
}
```

### REST Assured Retry Helper

```java
public class RetryHelper {
    public static Response executeWithRetry(Supplier<Response> apiCall) {
        int retryCount = Integer.parseInt(
            ConfigReader.getProperty("RETRY_COUNT", "3")
        );
        int retryDelay = Integer.parseInt(
            ConfigReader.getProperty("RETRY_DELAY", "1000")
        );
        
        Response response = null;
        Exception lastException = null;
        
        for (int i = 0; i <= retryCount; i++) {
            try {
                response = apiCall.get();
                if (response.getStatusCode() < 500) {
                    return response;
                }
            } catch (Exception e) {
                lastException = e;
                LoggerUtil.warn("Attempt " + (i + 1) + " failed: " + e.getMessage());
            }
            
            if (i < retryCount) {
                try {
                    Thread.sleep(retryDelay);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            }
        }
        
        if (lastException != null) {
            throw new RuntimeException("All retry attempts failed", lastException);
        }
        return response;
    }
}
```

---

## Reporting Configuration

### 1. Extent Reports Setup

**extent.properties:**
```properties
extent.reporter.spark.start=true
extent.reporter.spark.out=reports/extent-reports/ExtentReport.html
extent.reporter.spark.config=src/test/resources/extent-config.xml

screenshot.dir=reports/screenshots/
screenshot.rel.path=../screenshots/

extent.reporter.pdf.start=false
extent.reporter.json.start=false
```

**Extent Report Hook:**
```java
public class ExtentReportManager {
    private static ExtentReports extent;
    private static ThreadLocal<ExtentTest> test = new ThreadLocal<>();
    
    public static void initReport() {
        ExtentSparkReporter spark = new ExtentSparkReporter(
            "reports/extent-reports/ExtentReport.html"
        );
        extent = new ExtentReports();
        extent.attachReporter(spark);
        extent.setSystemInfo("Environment", ConfigReader.getProperty("BASE_URL"));
        extent.setSystemInfo("User", System.getProperty("user.name"));
    }
    
    public static void createTest(String testName) {
        ExtentTest extentTest = extent.createTest(testName);
        test.set(extentTest);
    }
    
    public static void logPass(String message) {
        test.get().pass(message);
    }
    
    public static void logFail(String message) {
        test.get().fail(message);
    }
    
    public static void flush() {
        extent.flush();
    }
}
```

### 2. Allure Reports Integration

**pom.xml additions:**
```xml
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
```

**Allure REST Assured Filter:**
```java
import io.qameta.allure.restassured.AllureRestAssured;

given()
    .filter(new AllureRestAssured())
    .get("/endpoint");
```

### 3. TEST_SUMMARY.md Generation

```java
public class TestSummaryGenerator {
    public static void generateSummary(List<TestResult> results) {
        StringBuilder summary = new StringBuilder();
        summary.append("# Test Execution Summary\n\n");
        summary.append("**Date:** ").append(LocalDateTime.now()).append("\n\n");
        
        long passed = results.stream().filter(r -> r.isPassed()).count();
        long failed = results.stream().filter(r -> r.isFailed()).count();
        long skipped = results.stream().filter(r -> r.isSkipped()).count();
        
        summary.append("## Overall Results\n\n");
        summary.append("- **Total Tests:** ").append(results.size()).append("\n");
        summary.append("- **Passed:** ").append(passed).append("\n");
        summary.append("- **Failed:** ").append(failed).append("\n");
        summary.append("- **Skipped:** ").append(skipped).append("\n\n");
        
        // Write to file
        Files.write(Paths.get("TEST_SUMMARY.md"), summary.toString().getBytes());
    }
}
```

---

## Maven Dependencies (pom.xml)

```xml
<properties>
    <java.version>17</java.version>
    <maven.compiler.source>17</maven.compiler.source>
    <maven.compiler.target>17</maven.compiler.target>
    <rest-assured.version>5.3.2</rest-assured.version>
    <cucumber.version>7.14.0</cucumber.version>
    <testng.version>7.8.0</testng.version>
    <jackson.version>2.15.3</jackson.version>
    <extent.version>5.1.1</extent.version>
    <allure.version>2.24.0</allure.version>
</properties>

<dependencies>
    <!-- REST Assured -->
    <dependency>
        <groupId>io.rest-assured</groupId>
        <artifactId>rest-assured</artifactId>
        <version>${rest-assured.version}</version>
    </dependency>
    <dependency>
        <groupId>io.rest-assured</groupId>
        <artifactId>json-schema-validator</artifactId>
        <version>${rest-assured.version}</version>
    </dependency>
    
    <!-- Cucumber -->
    <dependency>
        <groupId>io.cucumber</groupId>
        <artifactId>cucumber-java</artifactId>
        <version>${cucumber.version}</version>
    </dependency>
    <dependency>
        <groupId>io.cucumber</groupId>
        <artifactId>cucumber-testng</artifactId>
        <version>${cucumber.version}</version>
    </dependency>
    
    <!-- TestNG -->
    <dependency>
        <groupId>org.testng</groupId>
        <artifactId>testng</artifactId>
        <version>${testng.version}</version>
    </dependency>
    
    <!-- Jackson for JSON -->
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>${jackson.version}</version>
    </dependency>
    
    <!-- Extent Reports -->
    <dependency>
        <groupId>com.aventstack</groupId>
        <artifactId>extentreports</artifactId>
        <version>${extent.version}</version>
    </dependency>
    
    <!-- Allure -->
    <dependency>
        <groupId>io.qameta.allure</groupId>
        <artifactId>allure-cucumber7-jvm</artifactId>
        <version>${allure.version}</version>
    </dependency>
    <dependency>
        <groupId>io.qameta.allure</groupId>
        <artifactId>allure-rest-assured</artifactId>
        <version>${allure.version}</version>
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
    
    <!-- AssertJ for fluent assertions -->
    <dependency>
        <groupId>org.assertj</groupId>
        <artifactId>assertj-core</artifactId>
        <version>3.24.2</version>
    </dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.11.0</version>
            <configuration>
                <source>${java.version}</source>
                <target>${java.version}</target>
            </configuration>
        </plugin>
        
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

## Cucumber Feature File Best Practices

### Feature File Structure

```gherkin
Feature: Store API - Order Management
  As an API consumer
  I want to perform CRUD operations on store orders
  So that I can manage the pet store inventory

  Background:
    Given the API base URL is configured
    And authentication is set up

  @smoke @post
  Scenario: Create a new order successfully
    Given I have a valid order request payload
    When I send a POST request to "/store/order"
    Then the response status code should be 201
    And the response should match the order schema
    And the response should contain order ID
    And the response field "status" should equal "placed"

  @regression @get
  Scenario Outline: Retrieve order by ID
    When I send a GET request to "/store/order/<orderId>"
    Then the response status code should be <statusCode>
    And the response should match the order schema
    
    Examples:
      | orderId | statusCode |
      | 1       | 200        |
      | 2       | 200        |
      | 999999  | 404        |

  @negative
  Scenario: Create order with invalid data
    Given I have an order request with negative quantity
    When I send a POST request to "/store/order"
    Then the response status code should be 400
    And the response should contain error message
```

### Step Definition Implementation

**Best Practice**: Use TestContext for sharing state between steps (thread-safe for parallel execution):

```java
public class StoreSteps {
    private TestContext context;
    
    // Constructor injection for PicoContainer
    public StoreSteps(TestContext context) {
        this.context = context;
    }
    
    @Given("I have a valid order request payload")
    public void i_have_a_valid_order_request_payload() {
        Map<String, Object> requestBody = TestDataLoader.getRequestData("store/order");
        context.setRequestBody(requestBody);
    }
    
    @When("I send a POST request to {string}")
    public void i_send_a_post_request_to(String endpoint) {
        Response response = given()
            .spec(RequestSpecFactory.getAuthRequestSpec())
            .body(context.getRequestBody())
        .when()
            .post(endpoint);
        
        context.setResponse(response);
    }
    
    @Then("the response status code should be {int}")
    public void the_response_status_code_should_be(Integer statusCode) {
        context.getResponse().then()
            .statusCode(statusCode);
    }
    
    @Then("the response should match the order schema")
    public void the_response_should_match_the_order_schema() {
        context.getResponse().then()
            .body(matchesJsonSchemaInClasspath("schemas/order-response-schema.json"));
    }
    
    @Then("the response field {string} should equal {string}")
    public void the_response_field_should_equal(String field, String expectedValue) {
        context.getResponse().then()
            .body(field, equalTo(expectedValue));
    }
}
```

---

## Hooks and Base Setup

### Cucumber Hooks Implementation

```java
public class Hooks {
    private TestContext context;
    
    public Hooks(TestContext context) {
        this.context = context;
    }
    
    @Before
    public void setUp(Scenario scenario) {
        // Initialize REST Assured base configuration
        RestAssured.baseURI = ConfigReader.getProperty("BASE_URL");
        RestAssured.useRelaxedHTTPSValidation();
        
        // Set default timeout
        RestAssured.config = RestAssured.config()
            .httpClient(HttpClientConfig.httpClientConfig()
                .setParam("http.connection.timeout", 
                    Integer.parseInt(ConfigReader.getProperty("REQUEST_TIMEOUT", "30000")))
                .setParam("http.socket.timeout", 
                    Integer.parseInt(ConfigReader.getProperty("REQUEST_TIMEOUT", "30000"))));
        
        // Initialize test context
        context.setScenario(scenario);
        
        LoggerUtil.info("Starting scenario: " + scenario.getName());
    }
    
    @After
    public void tearDown(Scenario scenario) {
        if (scenario.isFailed()) {
            // Attach request/response to report
            if (context.getResponse() != null) {
                scenario.attach(
                    context.getResponse().getBody().asString().getBytes(),
                    "application/json",
                    "Response Body"
                );
            }
        }
        
        // Reset REST Assured
        RestAssured.reset();
        
        LoggerUtil.info("Finished scenario: " + scenario.getName() + 
            " - Status: " + scenario.getStatus());
    }
    
    @Before("@no-auth")
    public void skipAuthentication() {
        // Use base spec without authentication for negative auth tests
        LoggerUtil.info("Skipping authentication for negative test");
    }
}
```

### TestContext Implementation

```java
public class TestContext {
    private Response response;
    private Map<String, Object> requestBody;
    private Scenario scenario;
    
    public Response getResponse() {
        return response;
    }
    
    public void setResponse(Response response) {
        this.response = response;
    }
    
    public Map<String, Object> getRequestBody() {
        return requestBody;
    }
    
    public void setRequestBody(Map<String, Object> requestBody) {
        this.requestBody = requestBody;
    }
    
    public Scenario getScenario() {
        return scenario;
    }
    
    public void setScenario(Scenario scenario) {
        this.scenario = scenario;
    }
}
```

---

## Common Cucumber Step Definitions

```java
public class CommonSteps {
    private TestContext context;
    
    public CommonSteps(TestContext context) {
        this.context = context;
    }
    
    @Given("the API base URL is configured")
    public void the_api_base_url_is_configured() {
        // Already configured in Hooks
        LoggerUtil.info("Base URL: " + ConfigReader.getProperty("BASE_URL"));
    }
    
    @Given("authentication is set up")
    public void authentication_is_set_up() {
        // Authentication handled in RequestSpecFactory
        LoggerUtil.info("Authentication type: " + 
            ConfigReader.getProperty("AUTH_TYPE"));
    }
    
    @Then("the response should contain {string}")
    public void the_response_should_contain(String key) {
        context.getResponse().then()
            .body(key, notNullValue());
    }
    
    @Then("the response time should be less than {int} milliseconds")
    public void the_response_time_should_be_less_than_milliseconds(Integer maxTime) {
        context.getResponse().then()
            .time(lessThan(maxTime.longValue()));
    }
    
    @Then("the response should have status {string}")
    public void the_response_should_have_status(String status) {
        context.getResponse().then()
            .body("status", equalTo(Boolean.parseBoolean(status)));
    }
    
    @Then("the response should contain expected values for {string}")
    public void the_response_should_contain_expected_values(String endpoint) {
        Map<String, Object> expectedValues = TestDataLoader.getExpectedInResponse(endpoint);
        Response response = context.getResponse();
        
        expectedValues.forEach((key, expectedValue) -> {
            Object actualValue = response.jsonPath().get(key);
            assertThat(actualValue)
                .as("Validation failed for field: " + key)
                .isEqualTo(expectedValue);
        });
    }
}
```

---

## testng.xml Configuration

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE suite SYSTEM "http://testng.org/testng-1.0.dtd">
<suite name="API Test Suite" parallel="false" thread-count="1">
    <listeners>
        <listener class-name="com.api.automation.listeners.ExtentReportListener"/>
        <listener class-name="com.api.automation.listeners.TestRetryListener"/>
    </listeners>
    
    <test name="API Regression Tests">
        <classes>
            <class name="com.api.automation.runners.TestRunner"/>
        </classes>
    </test>
</suite>
```

---

## Failure Categorization

| Category | Definition | Action |
|----------|-----------|--------|
| Positive Scenario Failures | Swagger 200/201/204 that failed | CRITICAL - API bugs |
| Wrong Error Code | 500 instead of 404, or unexpected status | Document in TEST_SUMMARY.md |
| Schema Validation Failures | Response structure mismatch | CRITICAL - API contract violation |

---

## Critical Rules

| ❌ NEVER | ✅ ALWAYS |
|----------|----------|
| Modify assertions to match API behavior | Let tests fail when API is wrong |
| Skip failed assertions | Stick to Swagger definitions |
| Change expected status codes | Document failures in TEST_SUMMARY.md |
| Hardcode payloads in step definitions | Use `TestDataLoader.getRequestData()` for data-driven testing |
| Auto-generate optional fields (causes API issues) | Only auto-generate REQUIRED fields per Swagger |
| Create 100+ line step definitions with if-else | Keep step definitions clean (5-15 lines) |
| Mark failing tests as passing | Perform schema validation on responses |

---

## Best Practices Checklist

When generating a new test suite from scratch, ensure:

- [ ] **CRITICAL**: Feature file endpoint paths exactly match actual API endpoints (don't invent paths)
- [ ] **CRITICAL**: testData.json keys match feature file endpoints with `/api/` prefix removed
- [ ] **CRITICAL**: Parse Swagger response schemas to extract actual field names (response/data/result) - don't assume
- [ ] **CRITICAL**: Verify response field names from schema `properties` before writing assertions
- [ ] Verify all endpoint paths against API documentation or existing testData.json before creating features
- [ ] `testData.json` has proper structure with `endpoints` section
- [ ] Step definitions use `TestDataLoader.getRequestData(endpoint)` - no hardcoded payloads
- [ ] Only business-critical fields defined in endpoint `request` objects
- [ ] Three-level validation implemented (schema + types + key values)
- [ ] All helpers configured: `TestDataLoader.java`, `SchemaValidator.java`, `BaseAPIHelper.java`
- [ ] SSL certificate handling (`useRelaxedHTTPSValidation()`) configured
- [ ] RequestSpecification and ResponseSpecification factories created
- [ ] Proper logging configured (SLF4J + Logback)
- [ ] Retry logic implemented for flaky tests
- [ ] Multiple report formats configured (Cucumber, Extent, Allure)

---

## Execution Workflow

When agent is invoked:

1. **Read Configuration**:
   - Read `.env` file for `BASE_URL`, `AUTH_TYPE`, authentication credentials (JWT_TOKEN, JWT_USER_ID, etc.)

2. **Parse API Specification**:
   - Fetch/read specification from configured location
   - Validate Swagger 2.0 or OpenAPI 3.0 format
   - Extract endpoints, operations, request/response schemas, authentication
   - **Apply Endpoint Filtering**:
     - If `ENDPOINT_FILTER=all`: Generate tests for all endpoints in the API specification
     - If `ENDPOINT_FILTER=config`: Read `ENDPOINT_CONFIG_PATH` file and generate tests ONLY for endpoints where `enabled: true`
     - Match endpoints by HTTP method and path (e.g., `POST /api/favourites/v1/save`)
     - Skip any endpoints not listed in the config or marked `enabled: false`
   
   - **CRITICAL: Parse Response Schemas for Field Names**:
     - For each endpoint's success response (200/201), locate the response schema
     - Navigate to `responses` → `200` → `content` → `application/json` → `schema` → `$ref`
     - Resolve the `$ref` to find the actual schema definition in `components.schemas`
     - Extract top-level `properties` field names from the schema
     - Example from Swagger:
       ```json
       "ResponseMapStringListSalesPlanListingResponse": {
         "type": "object",
         "properties": {
           "response": { "type": "object" },    // ← Use this field name!
           "status": { "type": "boolean" },
           "error": { "$ref": "#/components/schemas/ErrorResponse" }
         }
       }
       ```
     - Use extracted field names in feature file assertions:
       - If schema has `"response"` property → `And the response should contain "response"`
       - If schema has `"data"` property → `And the response should contain "data"`
       - If schema has `"result"` property → `And the response should contain "result"`
     - **DO NOT assume/hardcode field names** - always extract from actual schema
     - **Fallback**: If schema parsing fails or no schema defined, default to `"response"` (this API's convention)

3. **Generate Test Suite**:
   - **testData.json**: Create centralized test data with `endpoints` sections
   - Gherkin feature files with positive/negative scenarios
   - **Data-driven step definitions**: Use `TestDataLoader.getRequestData(endpoint)` (5-15 lines each)
   - API helpers, schema validator, hooks, TestContext
   - Configuration files (pom.xml, testng.xml, .env.example)

4. **Execute Tests**:
   - Run all tests using `mvn clean test`
   - Apply retry logic (3 attempts per scenario)
   - Allow tests to fail naturally when API behavior differs from specification - Do NOT modify assertions

5. **Generate Reports**:
   - Cucumber HTML/JSON reports (auto-generated during test run)
   - Extent Reports dashboard
   - Generate Allure report using `mvn allure:report`
   - Create TEST_SUMMARY.md with comprehensive failure categorization

6. **Document Results**:
   - Categorize failures: Critical, Validation Gaps, Wrong Error Codes, Schema Violations
   - Provide clear next steps and prioritized action items

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| SSL certificate error | Add `RestAssured.useRelaxedHTTPSValidation()` in hooks or base spec |
| `404 Not Found` on valid endpoints | **CRITICAL**: Verify feature file endpoint paths exactly match testData.json keys (with `/api/` prefix). Don't invent endpoint names - use exact paths from API docs |
| `JSON path data doesn't match` / Field not found | **CRITICAL**: Parse Swagger response schema to get actual field name. Steps: 1) Find endpoint in `paths`, 2) Get `responses.200.content['application/json'].schema.$ref`, 3) Resolve to `components.schemas.SchemaName.properties`, 4) Use actual property names (e.g., "response", "data", "result") in assertions |
| Wrong field name assumed (data vs response) | **Always parse Swagger schema**. Example: If schema shows `"properties": {"response": {...}}`, use `And the response should contain "response"` NOT `"data"` |
| `404 Not Found` / `ApplicationNotFound` | Ensure `BASE_URL` includes full service path and check basePath configuration |
| Connection timeout | Increase `REQUEST_TIMEOUT` in .env file |
| JSON serialization error | Verify POJO classes have proper getters/setters and Jackson annotations |
| Schema validation failures | Check schema files are in correct location (`src/test/resources/schemas/`) |
| TestNG execution issues | Verify testng.xml is properly configured with correct test class paths |
| Maven dependency conflicts | Run `mvn dependency:tree` to identify and resolve conflicts |
| Auto-generation not working | Ensure Swagger JSON/URL is accessible and has valid schema definitions |
| Step definition not found | Check package paths in `@CucumberOptions` glue parameter |
| PicoContainer injection fails | Add `io.cucumber:cucumber-picocontainer` dependency to pom.xml |

---

## Execution Commands

```bash
# Run all tests
mvn clean test

# Run specific tags
mvn clean test -Dcucumber.filter.tags="@smoke"

# Run with specific environment
mvn clean test -Denv=staging

# Generate Allure report
mvn allure:serve

# Generate Allure report (static)
mvn allure:report

# Run with parallel execution
mvn clean test -Ddataproviderthreadcount=3
```

---

## Logging Configuration (logback.xml)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    
    <appender name="FILE" class="ch.qos.logback.core.FileAppender">
        <file>logs/test-execution.log</file>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    
    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="FILE"/>
    </root>
</configuration>
```

---

## Best Practices Summary

### 1. Test Organization
- One feature file per API endpoint group
- Reusable step definitions in common steps
- Background section for setup steps
- Use tags for categorization (@smoke, @regression, @negative)

### 2. Data Management
- Centralized test data in JSON format
- Environment-specific configurations in properties files
- POJO models for type safety
- Builder pattern for complex request objects

### 3. Validation Strategy
- Schema validation for structure
- JsonPath for field-level validation
- AssertJ for fluent assertions
- Custom validators for business rules

### 4. Error Handling
- Retry mechanism for flaky tests
- Comprehensive logging
- Screenshot/request-response capture on failure
- Proper exception handling

### 5. Reporting
- Multiple report formats (Cucumber, Extent, Allure)
- Test summary generation
- Trend analysis with Allure
- CI/CD integration ready

---

## Completion Criteria

- ✅ All test scenarios execute successfully
- ✅ Three-level validation (schema, data type, key values) implemented
- ✅ Retry logic configured for stability
- ✅ Multiple report formats generated
- ✅ Centralized configuration and test data
- ✅ Flexible authentication support
- ✅ Proper logging and debugging capabilities
- ✅ CI/CD ready with Maven integration
- ✅ Code follows Java and REST Assured best practices
- ✅ Comprehensive documentation in place

---

**Version**: 2.2 | **Updated**: December 16, 2025 | **Major Features**: Smart Auto-Generation from Swagger, Data-Driven Testing, Three-Level Validation
