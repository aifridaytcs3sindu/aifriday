# API Test Generator

**Description**: Automatically generates complete Playwright + Cucumber API test suites from Swagger/OpenAPI specifications with schema validation, flexible authentication (Basic, Bearer, JWT), and comprehensive reporting.

**Usage**: 
```
Generate API tests
```

or with explicit reference:

```
@workspace #file:copilot-instructions.md Generate API tests
```

---

## Agent Instructions

### Core Objective

Generate a **complete Playwright + Cucumber API test suite in JavaScript** for all endpoints defined in the API specification configured in the `.env` file, with:
- Schema validation using Ajv
- Flexible authentication (basic/bearer/jwt/none)
- Multiple report formats (Cucumber HTML/JSON, Allure dashboard, TEST_SUMMARY.md)
- Automatic retry logic (3 attempts per scenario)

### Configuration Source

Read all configuration from `.env` file:
- `INPUT_TYPE`: API specification format (`swagger_json`, `swagger_url`, `openapi_json`, `openapi_url`)
- `INPUT_PATH`: Location of API specification (file path or URL)
- `BASE_URL`: API base URL endpoint
- `AUTH_TYPE`: Authentication method (`basic`, `bearer`, `jwt`, `none`)
- `AUTH_USERNAME` / `AUTH_PASSWORD`: For Basic Auth
- `AUTH_TOKEN`: For Bearer Token Auth
- `JWT_TOKEN`, `JWT_USER_ID`, `JWT_APP_ID`, `JWT_USER_TEAM`: For JWT Auth
- `ENDPOINT_FILTER`: Endpoint filtering mode (`all` or `config`)
- `ENDPOINT_CONFIG_PATH`: Path to endpoint configuration file (when `ENDPOINT_FILTER=config`)
- `REQUEST_TIMEOUT`: API request timeout in milliseconds (default: 30000)

### Technical Stack

- **Node.js v18+** (ESM modules)
- **Playwright v1.45+** (APIRequestContext)
- **Cucumber.js v10+** (BDD framework with retry)
- **Chai v5+** (assertions)
- **Ajv v8+** (JSON Schema validation)
- **Allure** (allure-commandline, allure-js-commons)
  - **Note**: `allure-cucumberjs` reporter doesn't work properly with ESM modules. Use `generate-allure.js` script to convert Cucumber JSON to Allure format.
- **dotenv** (environment configuration)

---

## Important Implementation Details

### SSL Certificate Handling

Corporate environments often use self-signed certificates or proxies that cause SSL errors. Always configure Playwright to ignore HTTPS errors:

```javascript
// In world.js - init() and initWithoutAuth() methods
this.context = await request.newContext({
  baseURL: this.baseUrl,
  timeout: this.timeout,
  ignoreHTTPSErrors: true,  // REQUIRED for corporate environments
  extraHTTPHeaders: {
    'Content-Type': 'application/json',
    ...authHeaders
  }
});
```

### URL Path Construction

Playwright's `baseURL` behavior requires careful path handling:

1. **BASE_URL with trailing slash**: Ensure `baseUrl` ends with `/` in `world.js`:
```javascript
// In world.js constructor
let baseUrl = process.env.BASE_URL || '';
if (!baseUrl.endsWith('/')) {
  baseUrl = baseUrl + '/';
}
this.baseUrl = baseUrl;
```

2. **Endpoint paths without leading slash**: Remove leading `/` from endpoint paths in API helpers:
```javascript
// In baseApiHelper.js - makeRequest()
let url = endpoint.startsWith('/') ? endpoint.substring(1) : endpoint;
```

**Why this matters**: 
- If `baseURL = "https://api.example.com/service"` and endpoint = `/api/endpoint`
- Playwright treats leading `/` as absolute path from host root
- Result: `https://api.example.com/api/endpoint` (WRONG - missing `/service`)
- Fix: `baseURL = "https://api.example.com/service/"` + `api/endpoint` = correct URL

### Request/Response Logging

Include console logging in API helpers for debugging:

```javascript
// In baseApiHelper.js - makeRequest()
console.log(`\n  📡 ${method} ${url}`);
if (data) {
  console.log(`  📤 Request Body: ${JSON.stringify(data)}`);
}
console.log(`  📥 Response Status: ${statusCode}`);
if (responseBody) {
  const bodyStr = typeof responseBody === 'string' ? responseBody : JSON.stringify(responseBody);
  console.log(`  📥 Response Body: ${bodyStr.substring(0, 200)}${bodyStr.length > 200 ? '...' : ''}`);
}
```

---

## Project Structure

```
features/
├── step_definitions/
│   ├── common_steps.js           # Common step definitions (auth, status checks)
│   ├── post_steps.js             # POST request step definitions
│   ├── get_steps.js              # GET request step definitions
│   └── <endpoint>_steps.js       # Endpoint-specific step definitions
├── support/
│   ├── world.js                  # Playwright context initialization
│   └── hooks.js                  # Setup and teardown hooks
├── <endpoint1>.feature           # Gherkin scenarios for endpoint1
└── <endpoint2>.feature           # Gherkin scenarios for endpoint2
helpers/
├── baseApiHelper.js              # Base API helper with request/response attachments
├── testDataLoader.js             # Test data JSON loader (ESM compatible)
├── retryHelper.js                # Retry logic utilities
└── schemaValidator.js            # JSON Schema validation (Ajv)
test-data/
└── testData.json                 # Centralized test data (key-value pairs)
reports/                          # Generated test reports
├── cucumber-report.html
└── cucumber-report.json
allure-results/                   # Allure test results (generated)
allure-report/                    # Allure HTML report (generated)
.env                              # Environment configuration
.env.example                      # Environment template
cucumber.js                       # Cucumber configuration (retry: 3, parallel: 1)
playwright.config.js              # Playwright configuration
generate-allure.js                # Cucumber JSON to Allure converter
package.json                      # Dependencies and scripts
TEST_SUMMARY.md                   # Test execution summary (generated)
```

**Naming Convention**: Use lowercase filenames (`store_steps.js`, `storeApiHelper.js`) and camelCase function names (`createOrder()`, `getOrderById()`).

---

## Test Data Management

### Centralized Test Data

**Location**: `test-data/testData.json`

**Purpose**: Provides reusable test data values that GHCP can reference when generating tests, avoiding hardcoded values in test code.

**Structure**: Simple key-value pairs for individual parameters

```json
{
  "franchiseNumber": 1073741824,
  "orderId": "ORD-12345",
  "userId": "testuser123",
  "status": "COMPLETED"
}
```

### Usage Guidelines

**Smart Value Resolution Strategy**:

When generating request bodies, GHCP should follow this priority:

1. **Check `testData.json` first** - If a request body field matches a key in `testData.json`, use that value
2. **Generate random/default values** - If no match found in `testData.json`, auto-generate appropriate values based on Swagger schema:
   - Strings: Generate random strings or use Swagger examples
   - Numbers: Use appropriate numeric values or Swagger examples
   - Arrays: Generate sample arrays
   - Objects: Generate nested structures as needed

**Import Pattern**:
Use the `testDataLoader` helper to load test data (avoids Node.js JSON import compatibility issues):
```javascript
import { testData } from '../../helpers/testDataLoader.js';
```

**What to Store in `testData.json`**:
- **Critical values requiring real/specific data** (e.g., valid franchise numbers, existing order IDs)
- Status codes and enum values
- Date ranges and timestamps
- User credentials (non-sensitive test data only)
- Any field where you need control over the exact value

**What NOT to Store**:
- Fields that can be random (GHCP will auto-generate)
- Complete request body objects
- Sensitive production data
- Complex nested objects

---

## Test Coverage Requirements

### Positive Scenarios (200/201/204)
- Validate successful operations per Swagger/OpenAPI definitions
- **Schema validation** using Ajv (`helpers/schemaValidator.js`):
  - **GET (200)**: ALWAYS validate response schema
  - **POST/PUT (200/201)**: Validate IF response returns an object (skip simple messages)
  - **DELETE (204)**: No validation needed
  - Checks: required fields, data types, nested structures, array items
- Use status codes defined in specification
- If test fails → indicates API bug (not test bug)

### Negative Scenarios

Generate negative test cases based on HTTP methods:

| HTTP Method | Tested Negative Codes |
|-------------|-----------------------|
| **GET**     | 401, 404, 500         |
| **POST**    | 400, 401, 404, 500    |
| **PUT**     | 400, 404, 500         |
| **DELETE**  | 400, 404, 500         |

**Important**: If Swagger defines specific negative codes, use those instead.

**Expected Failures**:
- API returns 200 instead of 400 → Validation Gap (missing input validation)
- API returns 500 instead of 400/404 → Wrong Error Code (poor error handling)

---

## Authentication Handling

Read `AUTH_TYPE` from `.env` and configure accordingly:

**AUTH_TYPE=basic**:
- Use `AUTH_USERNAME` and `AUTH_PASSWORD`
- Encode as `Authorization: Basic <base64>`

**AUTH_TYPE=bearer**:
- Use `AUTH_TOKEN`
- Set header `Authorization: Bearer <token>`

**AUTH_TYPE=jwt**:
- Use `JWT_TOKEN`, `JWT_USER_ID`, `JWT_APP_ID`, `JWT_USER_TEAM` from `.env`
- Set headers:
  - `Content-Type: application/json` (always included)
  - `X-Auth-Token: <JWT_TOKEN>`
  - `X-MS-IBT-User-ID: <JWT_USER_ID>`
  - `X-MS-APP-ID: <JWT_APP_ID>`
  - `X-User-Team: <JWT_USER_TEAM>`

**AUTH_TYPE=none**:
- No authentication headers

Apply authentication to all requests requiring it per Swagger security definitions.

**Testing 401 Scenarios**: 
Use `@no-auth` tag for cleaner testing without authentication setup/removal steps:

- **Feature files**: Tag 401 scenarios with `@no-auth`, keep Background minimal (only `Given the API base URL is configured`)
- **Before hook** (`features/support/hooks.js`): Check for `@no-auth` tag in `pickle.tags`, call `initWithoutAuth()` instead of `init()`
- **World.js**: Implement `initWithoutAuth()` method that disposes any existing context (clears cache), creates fresh `request.newContext()` with only `Content-Type` header (no Authorization), and initializes API helpers
- **Other scenarios**: Add `Given authentication is set up` step explicitly to scenarios requiring authentication

Example:
```gherkin
Feature: Order API
  Background:
    Given the API base URL is configured

  Scenario: Successfully create order
    Given authentication is set up
    When I send a POST request to create order
    Then the response status code should be 200

  @no-auth
  Scenario: Create order with unauthorized access
    # No auth setup - @no-auth tag triggers initWithoutAuth()
    When I send a POST request to create order
    Then the response status code should be 401
```

---

## Configuration Files

⚠️ **CRITICAL**: Cucumber 10+ requires explicit `--format` flags in the package.json test script. The `format` settings in `cucumber.js` are not reliably applied.

### cucumber.js
```javascript
export default {
  default: {
    require: ['features/support/**/*.js', 'features/step_definitions/**/*.js'],
    format: [
      'progress-bar',
      'html:reports/cucumber-report.html',
      'json:reports/cucumber-report.json'
    ],
    formatOptions: {
      snippetInterface: 'async-await'
    },
    parallel: 1,  // Sequential execution (can be increased for parallel)
    retry: 3,     // Retry failed scenarios 3 times
    publishQuiet: true
  }
};
```

### package.json (scripts)
```json
{
  "type": "module",
  "scripts": {
    "test": "cucumber-js --format progress-bar --format html:reports/cucumber-report.html --format json:reports/cucumber-report.json",
    "test:watch": "nodemon --exec \"npm test\"",
    "generate:allure": "node generate-allure.js",
    "allure:report": "npm run generate:allure && allure generate allure-results --clean -o allure-report",
    "allure:open": "allure open allure-report",
    "test:allure": "npm test && npm run allure:report"
  }
}
```

**Important**: The `test` script must explicitly specify the `--format` options because Cucumber 10+ doesn't always apply the `format` settings from `cucumber.js` file. The explicit format options ensure:
- Progress bar output to console
- HTML report generation to `reports/cucumber-report.html`
- JSON report generation to `reports/cucumber-report.json` (required for Allure conversion)

### generate-allure.js

**Purpose**: Converts Cucumber JSON report to Allure format (required because allure-cucumberjs/reporter doesn't work properly with ESM + Cucumber 10+)

**Location**: Root directory (`generate-allure.js`)

**Implementation**: Reads `reports/cucumber-report.json` and generates Allure result files in `allure-results/` directory with proper test metadata, steps, timing, and status information.

---

## Reporting Requirements

Generate **three report formats**:

### 1. Cucumber HTML/JSON Reports
- Location: `/reports/cucumber-report.html`, `/reports/cucumber-report.json`
- Basic test execution summary

### 2. Allure Report (Primary)
- Location: `/allure-report/index.html`
- Interactive dashboard with test statistics, trends, categorization
- Open with: `npm run allure:open`

**Request/Response Attachments**:
All API helpers must attach request and response JSON to Allure reports for debugging and traceability.

**Implementation Approach**:
Since `allure-js-commons` attachments don't work with ESM + Cucumber 10+, use Cucumber's built-in `this.attach()` method instead. The attachments flow through the Cucumber JSON report and are converted to Allure format by `generate-allure.js`.

**Step 1: Add attachment method in world.js**:
```javascript
// In world.js - CustomWorld class
async attachRequestResponse(method, url, requestBody, statusCode, responseBody) {
  try {
    // Attach Request Details
    const requestDetails = {
      method: method,
      url: url,
      body: requestBody || null
    };
    await this.attach(JSON.stringify(requestDetails, null, 2), 'application/json');

    // Attach Response Details
    const responseDetails = {
      statusCode: statusCode,
      body: responseBody || null
    };
    await this.attach(JSON.stringify(responseDetails, null, 2), 'application/json');
  } catch (e) {
    // Attachment failures shouldn't break tests
  }
}
```

**Step 2: Update baseApiHelper.js to accept world reference**:
```javascript
export class BaseApiHelper {
  constructor(context, world = null) {
    this.context = context;
    this.world = world; // Reference to Cucumber World for attachments
  }

  async makeRequest(method, endpoint, options = {}) {
    // ... make request ...
    
    // Attach request/response to Cucumber report (for Allure)
    if (this.world && this.world.attachRequestResponse) {
      await this.world.attachRequestResponse(method, url, data, statusCode, responseBody);
    }
    
    return { response, responseBody, statusCode };
  }
}
```

**Step 3: Pass world reference in step definitions**:
```javascript
// In step definitions - pass 'this' as second parameter
const helper = new BaseApiHelper(this.context, this);
```

**What Gets Attached**:
- ✅ Request Details (method, URL, body as JSON)
- ✅ Response Details (statusCode, body as JSON)

**Benefits**:
- Full request/response visibility in Allure report
- Easy debugging of failed tests
- Complete API call traceability
- Attachments appear in "Attachments" tab for each test case
- Works with ESM modules and Cucumber 10+

### 3. TEST_SUMMARY.md
Comprehensive markdown summary with:
- Test Results (Total, Passed, Failed, Success Rate)
- Test Coverage by Endpoint (✅/❌/⚠️ status)
- **Failed Test Cases**:
  - **Positive Scenarios**: Swagger-defined success responses that failed
  - **Negative Scenarios**: Tests with wrong error codes
  - **Schema Validation Failures**: Response structure mismatches
- **Not Applicable Test Cases**: Tests failing due to missing API validation
- **API Issues Requiring Attention**: Critical vs Behavior Issues
- Execution Details, Steps, Duration
- Next Steps (prioritized action items)

---

## Failure Categorization

Document all failures in TEST_SUMMARY.md under these categories:

### Category 1: Positive Scenario Failures (CRITICAL)
- **Definition**: Swagger-defined success responses (200/201/204) that failed
- **Example**: POST expects 200 per Swagger but returns 500
- **Action**: Indicates API bugs - do NOT modify assertions

### Category 2: Negative Scenario Failures - Validation Gaps
- **Definition**: API returns 200 instead of 400 (missing input validation)
- **Documentation**: List in "Not Applicable Test Cases" section
- **Example**: Missing required parameter returns 200 instead of 400

### Category 3: Negative Scenario Failures - Wrong Error Code
- **Definition**: API returns wrong error code (500 instead of 400/404)
- **Documentation**: List in "Failed Test Cases → Negative Scenarios"
- **Example**: Invalid payload returns 500 instead of 400

### Category 4: Schema Validation Failures (CRITICAL)
- **Definition**: Response structure doesn't match API specification
- **Documentation**: List in "Failed Test Cases → Schema Validation Failures"
- **Examples**: Missing required fields, wrong data types, unexpected fields

---

## Critical Rules

### ❌ NEVER Do:
- Modify assertions to match actual API behavior
- Add graceful handling that skips failed assertions
- Change expected status codes based on API responses
- Add "# Note:" comments in feature files
- Mark tests as passing when they should fail
- Skip schema validation when it fails

### ✅ ALWAYS Do:
- Let tests fail when API behavior is wrong
- Stick to Swagger definitions for positive scenarios
- Document failures comprehensively in TEST_SUMMARY.md
- Categorize failures correctly (Critical vs Validation Gaps vs Schema Violations)
- Perform schema validation on GET responses (200)
- Perform schema validation on POST/PUT responses IF they return an object

---

## Execution Workflow

When agent is invoked:

1. **Read Configuration**:
   - Read `.env` for `INPUT_TYPE`, `INPUT_PATH`, `BASE_URL`, `AUTH_TYPE`, credentials

2. **Parse API Specification**:
   - Fetch/read specification from configured location
   - Validate Swagger 2.0 or OpenAPI 3.0 format
   - Extract endpoints, operations, request/response schemas, authentication
   - **Apply Endpoint Filtering**:
     - If `ENDPOINT_FILTER=all`: Generate tests for all endpoints in the API specification
     - If `ENDPOINT_FILTER=config`: Read `ENDPOINT_CONFIG_PATH` file and generate tests ONLY for endpoints where `enabled: true`
     - Match endpoints by HTTP method and path (e.g., `POST /api/favourites/v1/save`)
     - Skip any endpoints not listed in the config or marked `enabled: false`

3. **Generate Test Suite**:
   - Data models (JavaScript classes) for each endpoint
   - Gherkin feature files with positive/negative scenarios
   - Step definitions using Playwright APIRequestContext
   - API helpers with methods per operation
   - Schema validator using Ajv
   - Hooks and world.js for Playwright context
   - Allure converter script (generate-allure.js)
   - All required configuration files

4. **Execute Tests**:
   - Run all tests using `npm test`
   - Apply retry logic (3 attempts per scenario)
   - Allow tests to fail naturally when API behavior differs from specification - Do NOT modify assertions

5. **Generate Reports**:
   - Cucumber HTML/JSON reports (auto-generated during test run)
   - Convert Cucumber JSON to Allure format using `npm run generate:allure`
   - Generate Allure dashboard using `npm run allure:report`
   - Create TEST_SUMMARY.md with comprehensive failure categorization

6. **Document Results**:
   - Categorize failures: Critical, Validation Gaps, Wrong Error Codes, Schema Violations
   - Provide clear next steps and prioritized action items

---

## Troubleshooting

### Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| `unable to get local issuer certificate` | SSL/TLS certificate error | Add `ignoreHTTPSErrors: true` in `request.newContext()` |
| `404 Not Found` on valid endpoints | URL path construction issue | Ensure `baseUrl` ends with `/` and remove leading `/` from endpoints |
| `ApplicationNotFound` error from Apigee | Path not including base path | Check that `BASE_URL` includes full service path (e.g., `/dsOrchService`) |
| JSON import error with `assert { type: 'json' }` | Node.js ESM compatibility | Use `testDataLoader.js` helper with `fs.readFileSync` instead |
| `function timed out, ensure the promise resolves within 5000 milliseconds` | Cucumber step timeout too short | Add `setDefaultTimeout(60000)` in `hooks.js` |
| `'crypto' does not provide an export named 'v4'` | Wrong UUID import in generate-allure.js | Use `import crypto from 'crypto'` (not `{ v4 as uuidv4 }`) and custom `generateUUID()` function |
| Allure attachments not appearing | `allure-js-commons` doesn't work with ESM | Use Cucumber's `this.attach()` method in world.js and pass world reference to BaseApiHelper |

---

**Version**: 2.0 | **Updated**: November 2025
