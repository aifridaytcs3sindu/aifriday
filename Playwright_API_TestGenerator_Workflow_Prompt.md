# API Test Generator

**Description**: Automatically generates complete Playwright + Cucumber API test suites from Swagger/OpenAPI specifications with schema validation, flexible authentication (Basic, Bearer, JWT), and comprehensive reporting.

**Usage**: `Generate API tests` or `@workspace #file:copilot-instructions.md Generate API tests`

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
Always add `ignoreHTTPSErrors: true` in `request.newContext()` for corporate environments.

### URL Path Construction
- Ensure `baseUrl` ends with `/` in `world.js`
- Remove leading `/` from endpoint paths in API helpers
- This prevents Playwright from treating `/api/endpoint` as absolute path from host root

### Request/Response Logging
Include console logging in API helpers (`📡`, `📤`, `📥` prefixes) for debugging.

---

## Project Structure

```
features/
├── step_definitions/
│   ├── common_steps.js           # Common step definitions (auth, status checks, key values validation)
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
├── testDataLoader.js             # Test data loader with validation functions
├── retryHelper.js                # Retry logic utilities
└── schemaValidator.js            # JSON Schema validation (Ajv)
test-data/
└── testData.json                 # Centralized test data with request/expectedInResponse
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

### Key Helper Functions

**testDataLoader.js** provides the following functions:

| Function | Purpose | Returns |
|----------|---------|---------|
| `getRequestData(endpoint)` | Get request payload for an endpoint | Object with request fields |
| `getExpectedInResponse(endpoint)` | Get expected values for response validation | Object with expected field values |
| `validateExpectedValues(response, endpoint)` | Validate response against expected values | `{ valid: boolean, mismatches: Array }` |
| `getEndpointData(endpoint)` | Get merged common + request data | Object with all fields |
| `getCommonData()` | Get shared common data | Object with common fields |

**Naming Convention**: Use lowercase filenames (`store_steps.js`, `storeApiHelper.js`) and camelCase function names (`createOrder()`, `getOrderById()`).

---

## Test Data Management

### Centralized Test Data

**Location**: `test-data/testData.json`

**Purpose**: Provides reusable test data values organized by endpoint, enabling GHCP to reference specific data for each API endpoint without ambiguity. Also defines expected response values for key field validation.

**Structure**: Hierarchical structure with common data, endpoint-specific request data, and expected response values

```json
{
  "description": "Test data organized by endpoint with request data and expected response values for validation",
  
  "common": {
    "userId": "850680",
    "userType": "global",
    "franchiseeList": [8439, 8345, 5384, ...]
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
| **1. Schema Validation** | Response structure matches expected JSON schema | Ajv library validates against schemas in `schemas/` folder | Always for 200/201 responses |
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

**Dot Notation**: Use `response.fieldName` to validate nested fields in the response object.

### Usage Guidelines

**Import**: `import { getRequestData, getExpectedInResponse, validateExpectedValues } from '../../helpers/testDataLoader.js';`

**Value Resolution**: Endpoint-specific data → Common data → Random/default values

**Store in testData.json**: Endpoint-specific values, valid IDs, common shared data  
**Don't Store**: Sensitive data, complete objects, randomly-generated fields

---

## Test Coverage Requirements

### Positive Scenarios (200/201/204)
- Validate successful operations per Swagger/OpenAPI definitions
- **Three-Level Validation**:
  1. **Schema validation** using Ajv - validates response structure and required fields
  2. **Data type validation** - ensures fields have correct types (part of schema validation)
  3. **Key values validation** using `expectedInResponse` in `testData.json` - validates business-critical values
- Use status codes defined in specification
- If test fails → indicates API bug (not test bug)

### Feature File Pattern

```gherkin
@positive
Scenario: Successfully get overall budget listing
  Given authentication is set up
  When I send a POST request to "/api/optionsplanning/budget/overallbudget/listing" with valid payload
  Then the response status code should be 200
  And the response should have status true
  And the response should contain expected values for "optionsplanning/budget/overallbudget/listing"
```

### Negative Scenarios

**Always generate** 401 Unauthorized tests for all HTTP methods (GET, POST, PUT, DELETE) to ensure the API implements proper authentication, even if not documented in Swagger.

For other status codes (400, 403, 404), only generate test cases if documented in the Swagger/OpenAPI specification.

## Authentication Handling

| AUTH_TYPE | Variables | Headers |
|-----------|-----------|--------|
| `basic` | `AUTH_USERNAME`, `AUTH_PASSWORD` | `Authorization: Basic <base64>` |
| `bearer` | `AUTH_TOKEN` | `Authorization: Bearer <token>` |
| `jwt` | `JWT_TOKEN`, `JWT_USER_ID`, `JWT_APP_ID`, `JWT_USER_TEAM` | `X-Auth-Token`, `X-MS-IBT-User-ID`, `X-MS-APP-ID`, `X-User-Team` |
| `none` | - | No auth headers |

**401 Testing**: Use `@no-auth` tag → hooks call `initWithoutAuth()`

---

## Configuration Files

⚠️ **CRITICAL**: Cucumber 10+ requires explicit `--format` flags in package.json test script.

### Key Scripts
```json
{
  "test": "cucumber-js --format progress-bar --format html:reports/cucumber-report.html --format json:reports/cucumber-report.json",
  "allure:report": "npm run generate:allure && allure generate allure-results --clean -o allure-report",
  "test:allure": "npm test && npm run allure:report"
}
```### Negative Scenarios

401 Unauthorized tests are generated for all HTTP methods (GET, POST, PUT, DELETE) 
to ensure the API implements proper authentication, even if not documented in Swagger.

For other status codes (400, 403, 404), only generate test cases if documented in 
the Swagger/OpenAPI specification.

### generate-allure.js
Converts Cucumber JSON report to Allure format (required because allure-cucumberjs doesn't work with ESM + Cucumber 10+).

---

## Reporting Requirements

Generate **three report formats**:

| Report | Location | Purpose |
|--------|----------|---------|
| Cucumber HTML/JSON | `/reports/` | Basic test execution summary |
| Allure Report | `/allure-report/` | Interactive dashboard with trends |
| TEST_SUMMARY.md | Root directory | Comprehensive markdown summary |

**Request/Response Attachments**: Use Cucumber's `this.attach()` in world.js. Pass world reference to BaseApiHelper for automatic attachment.

---

## Failure Categorization

| Category | Definition | Action |
|----------|-----------|--------|
| Positive Scenario Failures | Swagger 200/201/204 that failed | CRITICAL - API bugs |
| Authentication Failures | 401 test returns different code | Document in TEST_SUMMARY.md |
| Schema Validation Failures | Response structure mismatch | CRITICAL - API contract violation |

---

## Critical Rules

| ❌ NEVER | ✅ ALWAYS |
|----------|----------|
| Modify assertions to match API behavior | Let tests fail when API is wrong |
| Skip failed assertions | Stick to Swagger definitions |
| Change expected status codes | Document failures in TEST_SUMMARY.md |
| Mark failing tests as passing | Perform schema validation on responses |

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

| Issue | Solution |
|-------|----------|
| SSL certificate error | Add `ignoreHTTPSErrors: true` in `request.newContext()` |
| `404 Not Found` on API calls | Ensure `BASE_URL` ends with `/` and includes full service path |
| Step timeout (5000ms) | `setDefaultTimeout(60000)` in `hooks.js` |
| JSON import error (ESM) | Use `testDataLoader.js` with `fs.readFileSync` instead of `assert { type: 'json' }` |

---

## README Generation

Generate a `README.md` file that includes:
- Project overview and purpose
- Prerequisites (Node.js version, dependencies)
- Installation steps (`npm install`)
- Environment configuration (`.env` setup with `endpoint-config.json`)
- How to run tests (`npm test`, `npm run test:tag @positive`)
- Project structure overview
- Reporting information (HTML reports location)
- Troubleshooting common issues

---

**Version**: 2.3 | **Updated**: December 2025
