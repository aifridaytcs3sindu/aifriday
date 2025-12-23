# =========================================================
# 🧠 Copilot Global Instructions for This Project
# =========================================================
You are a Senior QA Automation Architect with expertise in Playwright and Cucumber.
Always generate enterprise-ready, maintainable, and scalable automation.
Follow best practices, folder structure, environment handling, reporting, retries, and self-healing wherever possible.

# =========================================================
# 📊 TEST GENERATION FLOW
# =========================================================
```
┌─────────────────────────────────────────────────────────────────┐
│                          INPUT                                  │
├─────────────────────────────────────────────────────────────────┤
│  • User Journeys / Requirements (Excel, Markdown, plain text)  │
│  • copilot-instructions.md (this file)                         │
│  • Existing Page Objects & Step Definitions                    │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                       USER PROMPT                               │
├─────────────────────────────────────────────────────────────────┤
│  "Generate Playwright tests for [feature name]"                 │
│  "Create step definitions for login flow"                       │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                     AI PROCESSING                               │
├─────────────────────────────────────────────────────────────────┤
│  1. Read requirements & existing code                           │
│  2. Follow copilot-instructions.md rules                        │
│  3. Discover selectors from DOM (if needed)                     │
│  4. Generate code following project patterns                    │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                         OUTPUT                                  │
├─────────────────────────────────────────────────────────────────┤
│  • Feature File        →  features/[name].feature              │
│  • Step Definitions    →  steps/[name].steps.ts                │
│  • Page Objects        →  tests/pages/[Name]Page.ts            │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                       EXECUTION                                 │
├─────────────────────────────────────────────────────────────────┤
│  npx playwright test → HTML Report + Cucumber Report + JUnit   │
└─────────────────────────────────────────────────────────────────┘
```

# =========================================================
# 🚨 CRITICAL: DISCOVERY-FIRST APPROACH
# =========================================================
# NEVER generate test code based on assumptions about the application.
# ALWAYS run discovery tests FIRST to understand actual:
#   - Selectors (every app has unique class names)
#   - Interaction patterns (click vs checkbox, button vs icon)
#   - Wait times (Angular apps need 6+ seconds)
#   - Drag-and-drop mechanisms (dragTo vs native mouse)
#   - Save/submit patterns (button vs Enter key vs icon)
#   - Confirmation patterns (toast vs inline vs none)
#
# 🔄 AUTO-DISCOVERY ON FAILURE:
# If ANY test fails due to navigation or selector issues:
#   1. DO NOT just retry with same code
#   2. CREATE a discovery test to find actual selectors/navigation
#   3. EXTRACT real DOM elements and log them
#   4. UPDATE page objects with discovered values
#   5. RE-RUN the test with corrected code
#
# See Section 2️⃣-B for the mandatory 5-Phase Pre-Flight Discovery Protocol.
# See "Common Pitfalls and Anti-Patterns" section at the end.
# =========================================================

## To-Do List:
1. **🔍 Run Pre-Flight Discovery** (MANDATORY - see Section 2️⃣-B):
   - Phase 1: Authentication flow discovery
   - Phase 2: Navigation pattern discovery
   - Phase 3: Page element/selector discovery
   - Phase 4: Interaction pattern discovery
   - Phase 5: Drag-and-drop discovery (if applicable)
2. **Document Discovery Results** in YAML format before proceeding
3. **Create the project folder structure**: Ensure directories exist
4. **Create feature files** in `features/` - BDD scenarios (Gherkin)
5. **Implement step definitions** in `steps/` using DISCOVERED selectors (not assumed)
6. **Implement page objects** in `tests/pages/` with proper waits
7. **Set up hooks and fixtures** in `support/` as TypeScript files
8. **Execute tests**: Run `npm test` and verify all tests pass
9. **Fix failures immediately**: Debug and fix, don't just re-run
10. **Generate reports** and provide execution summary
# =========================================================

## Project Type
This project uses **playwright-bdd**: **Feature files (Gherkin)** + **Step definition files (.steps.ts)** with Playwright.

**Architecture:**
- **Feature files (.feature)**: BDD scenarios in Gherkin syntax - EXECUTED via playwright-bdd
- **Step files (.steps.ts)**: Step definitions that implement Given/When/Then steps
- **Page objects (.page.ts)**: Reusable page interaction classes
- **Benefits**: True BDD execution + Playwright's power + readable scenarios

Copilot should generate **feature files**, **step definition files**, and **page objects** for all test scenarios.

---

## 1️⃣ Objective
Whenever test scenarios or test cases are described, generate:
- `.feature` file (Gherkin syntax) - **Executable BDD scenarios**
- `.steps.ts` file (Step definitions) - **Implements Given/When/Then steps**
- Page object (`.page.ts`) for reusable actions in TypeScript

**Important:** 
- Feature files (.feature) are **executed** via playwright-bdd
- Step files (.steps.ts) contain the actual Playwright implementation
- Page objects encapsulate page interactions

All generated code must conform to this architecture and pass execution on the first run.

---

## 2️⃣ Folder & File Structure

Implement the following folder structure for your **Playwright with BDD documentation** project:

```
features/
└─ [feature-name].feature               # BDD scenarios (Gherkin) - EXECUTED via playwright-bdd
steps/
└─ [feature-name].steps.ts              # Step definitions implementing Given/When/Then
support/
├─ hooks.ts                             # Playwright test hooks (beforeEach, afterEach)
├─ fixtures.ts                          # Custom Playwright fixtures
├─ retry-helper.ts                      # Retry logic with exponential backoff
└─ helpers/                             # Folder for helper functions
tests/
└─ pages/                               # Page objects for tests
    ├─ BasePage.ts                      # Base class with common methods (retryAction, waitForPageLoad)
    └─ [PageName]Page.ts                # Page object file in TypeScript (e.g., LoginPage.ts)
reports/
├─ json/
│   └─ playwright-report.json           # JSON report from Playwright
├─ junit/
│   └─ junit.xml                        # JUnit XML report from Playwright
└─ playwright-report/                   # HTML interactive report (auto-generated by Playwright)
    └─ index.html
copilot-instructions.md                 # AI coding assistant instructions (this file)
.env                                    # Environment variables (LOCAL - add to .gitignore)
.env.example                            # Environment template (COMMIT this)
.gitignore                              # Git ignore file
playwright.config.ts                    # Playwright configuration with reporters
tsconfig.json                           # TypeScript configuration
package.json                            # NPM package manifest
requirements.xlsx                       # Test requirements from BA/QA
```

**Key Points:**
- **features/**: BDD scenarios in Gherkin - EXECUTED via playwright-bdd
- **steps/**: Step definitions implementing Given/When/Then steps
- **tests/pages/**: Page objects for reusable page interactions
- **reports/**: All reports generated by Playwright (HTML, JSON, JUnit)

---

## 2️⃣-A Environment Configuration

### `.env` File (Configuration Only - NO CREDENTIALS)

Create a `.env` file in the project root for **non-sensitive configuration only**:

```env
# Application Configuration
baseUrl=https://ds-dev.marksandspencer.app/

# Browser Settings (optional)
HEADLESS=false
SLOW_MO=0
```

### 🔐 Credentials (PowerShell Environment Variables - NOT in .env)

**Credentials MUST be set as PowerShell environment variables, NOT stored in .env file:**

```powershell
# Set credentials for current session
$env:DS_USERNAME='Y9215320@mnscorp.net'
$env:DS_PASSWORD='YourSecurePassword'

# Run tests (credentials are read from environment)
npx playwright test

# One-liner: Set credentials and run tests
$env:DS_USERNAME='Y9215320@mnscorp.net'; $env:DS_PASSWORD='YourPassword'; npx playwright test --grep "@all"
```

**Why credentials are NOT in .env:**
- ✅ `.env` can be committed safely (contains no secrets)
- ✅ Credentials never stored in project files
- ✅ CI/CD systems inject credentials as environment variables
- ✅ Different users use their own credentials
- ✅ Follows security best practices

**Important:**
- Use `DS_USERNAME` and `DS_PASSWORD` (not `username`/`password`) to avoid conflicts with Windows system environment variables
- Install dotenv for baseUrl: `npm install dotenv`

**Usage in code:**
```typescript
// In playwright.config.ts
import dotenv from 'dotenv';
dotenv.config();  // Loads baseUrl from .env

export default defineConfig({
  use: {
    baseURL: process.env.baseUrl || 'https://ds-dev.marksandspencer.app/',
  },
});

// In page objects - credentials come from PowerShell environment variables
export class LoginPage {
  async login(page: Page) {
    // DS_USERNAME and DS_PASSWORD are set via PowerShell, NOT .env
    const username = process.env.DS_USERNAME || '';
    const password = process.env.DS_PASSWORD || '';
    // REPLACE with actual selectors from Phase 1 discovery
    await page.fill('#discovered-username-selector', username);
    await page.fill('#discovered-password-selector', password);
  }
}
```

**Note:** Parallelization (`WORKERS`) and retry settings are configured directly in `playwright.config.ts`, not in `.env`. This keeps configuration separate from secrets.

---

## 2️⃣-B 🚨 CRITICAL: Pre-Flight Discovery Process (MANDATORY BEFORE GENERATING TESTS)

### Why Pre-Flight Discovery is Essential

**The #1 cause of test failures is generating code based on assumptions instead of actual application behavior.**

When generating tests, NEVER assume:
- Selector names or patterns (every app has unique class names)
- Interaction patterns (click vs checkbox, button vs Enter key)
- Page transitions (instant vs delayed, Angular apps need 3-6 seconds)
- Confirmation mechanisms (toast vs inline vs none)
- Drag-and-drop behavior (HTML5 vs custom implementation)
- Password field behavior (special characters may need special handling)

**ALWAYS run discovery tests first, document findings, then generate code that matches.**

### The 5-Phase Pre-Flight Discovery Protocol

Before writing ANY page object or step definition, execute these discovery phases:

| Phase | Purpose | Key Actions |
|-------|---------|-------------|
| **Phase 1: Authentication** | Discover login flow | Detect OAuth vs Standard, find form selectors |
| **Phase 2: Navigation** | Map navigation paths | Find nav elements, measure load times (6+ sec for Angular) |
| **Phase 3: Page Elements** | Extract real selectors | Get actual class names from DOM, not assumed ones |
| **Phase 4: Interactions** | Discover patterns | Click vs checkbox, save mechanism (button/icon/Enter) |
| **Phase 5: Drag-Drop** | Test drag methods | Try `dragTo()` first, then native mouse if needed |

**Generic Discovery Test Template:**
```typescript
test('Discover page elements @discovery', async ({ page }) => {
  await page.goto('/');
  await page.waitForLoadState('networkidle');
  await page.waitForTimeout(3000); // Wait for Angular/React
  
  // Extract all unique classes
  const classes = await page.evaluate(() => {
    return Array.from(new Set(
      Array.from(document.querySelectorAll('[class]'))
        .flatMap(el => el.className.split(' '))
        .filter(c => c.length > 3 && !c.includes('ng-'))
    ));
  });
  console.log('Classes:', classes);
  
  // Find interactive elements
  const clickables = await page.evaluate(() => {
    return Array.from(document.querySelectorAll('a, button, [role="button"]'))
      .slice(0, 15).map(el => ({ text: el.textContent?.trim().substring(0, 30), class: el.className }));
  });
  console.log('Clickables:', JSON.stringify(clickables, null, 2));
  
  await page.screenshot({ path: 'reports/discovery.png' });
});
```

**Document findings in YAML format before coding:**

After running discovery, document findings:

```yaml
# Discovery Results for [Application Name]
# Run discovery tests BEFORE generating page objects

authentication:
  type: OAuth  # OAuth | Standard | SSO
  provider: Microsoft Azure AD
  flow: email → next → password → submit → stay signed in
  password_method: evaluate  # fill | pressSequentially | evaluate (for special chars)
  selectors:
    email: input[type="email"]
    password: input[name="passwd"]
    submit: input[type="submit"]
    stay_signed_in: input[value="Yes"]
  post_login_url: /home

navigation:
  wait_required: 6 seconds  # Angular apps need this
  steps:
    - selector: .home-bu-label:has-text("CATEGORY")
    - wait: 2000ms
    - selector: .dept-list:has-text("Department")
    - wait: 1000ms
    - selector: .dept-go-next

page_elements:
  products:
    selector: .products-large  # NOT .product-tile
    load_time: 6+ seconds
    count: 50
    
interactions:
  selection:
    method: click  # NOT checkbox
    feedback: adds 'active' class
  save:
    method: icon click + Enter  # NOT button
    selector: .rhs-inp-list
    confirmation: check state (no toast)
  drag_drop:
    source: '#brc-items-new .products-small'
    target: '.br-actions'
    method: native mouse  # NOT dragTo
```

### Password Handling for Special Characters

**When passwords contain special characters (`^`, `>`, `}`, `*`, `|`, etc.):**

```typescript
// ❌ FAILS with special characters:
await passwordInput.fill('I3>9uvb^v?xD*}B}');
await passwordInput.pressSequentially('I3>9uvb^v?xD*}B}');

// ✅ WORKS with ALL characters:
await passwordInput.evaluate((el: HTMLInputElement, pwd: string) => {
  el.value = pwd;
  el.dispatchEvent(new Event('input', { bubbles: true }));
  el.dispatchEvent(new Event('change', { bubbles: true }));
}, password);
```

**Always use the `evaluate` method for password fields unless you're certain no special characters exist.**

---

## 3️⃣ Feature File Rules
- Use **Gherkin syntax** (`Feature:`, `Scenario:`, `Given/When/Then`).
- Each test case = one `Scenario`.
- Titles are descriptive and user-friendly.
- Scenarios must match natural steps from the user prompt.

### ⚠️ CRITICAL: Feature File Alignment with Step Definitions
**Feature files (.feature) are EXECUTED via playwright-bdd and MUST have corresponding step definitions (.steps.ts).**

**Rules:**
1. **Scenario Count**: Each scenario in a `.feature` file MUST have all steps implemented in the corresponding `.steps.ts` file.
   - Example: `login.feature` has 3 scenarios → `login.steps.ts` implements all Given/When/Then steps
2. **Step Matching**: Each Gherkin step MUST have a matching step definition.
3. **Tag Alignment**: Use the same tags (@smoke, @regression, @P0) in feature files.
4. **Step Accuracy**: Gherkin steps should reflect the actual implementation steps, not idealized flows.
   - ❌ BAD: "When I click the login button" (if OAuth is used)
   - ✅ GOOD: "When I enter my email address and click Next button" (reflects OAuth multi-step)
5. **Background Sections**: Use `Background:` for common preconditions shared across all scenarios.

**Validation Checklist (before marking implementation complete):**
- [ ] All scenarios have step definitions implemented
- [ ] Each Gherkin step has a matching step definition
- [ ] Tags are consistent across feature files
- [ ] Steps reflect actual implementation (not assumptions)

### Feature file partitioning
When the user requests multiple functional areas (e.g. Login, Cart, Checkout), **generate a separate `.feature` file for each area**. Do NOT combine unrelated features into a single `.feature` file.
Naming:
- Use `features/<feature-name>.feature` where `<feature-name>` is lowercase-hyphen (e.g. `login.feature`, `cart.feature`, `checkout.feature`).
- Group related Scenarios inside that file only.

---

## 4️⃣ Step Definition Rules (playwright-bdd)

* Create step definition files under `steps/` with descriptive names (e.g. `login.steps.ts`, `range-builder.steps.ts`, `product-browsing.steps.ts`).
* Use **TypeScript** syntax with proper typing.
* Import and use playwright-bdd's `createBdd()` pattern:
  ```typescript
  import { expect } from '@playwright/test';
  import { createBdd } from 'playwright-bdd';
  import { LoginPage } from '../tests/pages/LoginPage';

  const { Given, When, Then } = createBdd();

  let loginPage: LoginPage;

  Given('I navigate to the DS application', async ({ page }) => {
    loginPage = new LoginPage(page);
    await loginPage.navigate();
  });

  When('I enter my email address {string}', async ({ page }, email: string) => {
    await loginPage.fillUsername(email);
  });

  Then('I should be redirected to the Home page', async ({ page }) => {
    await page.waitForURL('**/home**', { timeout: 30000 });
  });
  ```
* Always prefix Playwright actions with `await`.
* Each step definition maps to a Gherkin step in the feature file.
* Handle waits with Playwright's built-in methods:
  * `page.waitForSelector(selector)`
  * `page.waitForURL(url)` or `await page.goto(url, { waitUntil: 'networkidle' })`
  * `await expect(page.locator(selector)).toBeVisible()`
* Each step performs a single logical action.
* Use retries for flaky operations (selectors, clicks, navigation).

---

## 5️⃣ Page Object Rules

* **Extend `BasePage`** for common functionality (retryAction, waitForPageLoad).
* Create page objects as TypeScript classes with proper typing.
* Accept a `Page` object in the constructor from Playwright.
* Each page object encapsulates actions and selectors for a specific page.

**Example:**
```typescript
import { Page, Locator } from '@playwright/test';
import { BasePage } from './BasePage';

export class LoginPage extends BasePage {
  private readonly emailInput: Locator;
  
  constructor(page: Page) {
    super(page);  // Pass page to BasePage
    this.emailInput = page.locator('input[type="email"]');
  }
}
```

### 🎯 CRITICAL: NEVER Assume Selectors - Always Extract from DOM

**THE GOLDEN RULE: All selectors MUST come from actual DOM inspection, never from assumptions.**

> ⚠️ **See Section 2️⃣-B for the complete 5-Phase Pre-Flight Discovery Protocol.**
> Run discovery tests BEFORE implementing any page objects.

**Quick Reference - Selector Discovery:**
```typescript
// ❌ WRONG (assumed selectors that don't exist):
this.productTiles = page.locator('.product-tile, .product-card');

// ✅ CORRECT (discovered from actual DOM):
this.productTiles = page.locator('.products-large');
```

**Framework-Specific Selector Patterns:**

| Framework | Typical Pattern | Example |
|-----------|----------------|---------|
| Angular | Component-based classes | `.products-large`, `.pro-lrg-img-wrap` |
| React | CSS Modules with hashes | `.Product_container__a3b4c` |
| Vue | Scoped classes | `[data-v-abc123]` |
| Plain CSS | BEM notation | `.product__tile--large` |

### 🔐 Authentication Detection (OAuth vs Standard Login)

> ⚠️ **See Section 2️⃣-B for complete authentication discovery protocol.**

**OAuth Detection Signs:**
- URL redirects to: `login.microsoftonline.com`, `accounts.google.com`, `okta.com`
- Multi-step flow: Email → Next → Password → Submit → Stay signed in?
- Use `evaluate()` method for passwords with special characters

**OAuth Implementation Key Points:**
1. Separate methods for each step (fillUsername, clickNext, fillPassword, clickSignIn)
2. Wait 2+ seconds after Next button for password page to load
3. Handle "Stay signed in?" prompt with try-catch (may not appear)
4. Verify final redirect to application URL (not OAuth provider)
5. Use generic selectors: `input[type="email"]`, `input[type="password"]`, `input[type="submit"]`

**Refer to `tests/pages/LoginPage.ts` in this project for the actual implementation.**

---

## 6️⃣ Hooks (Playwright Test Hooks):
- Use Playwright's native test hooks in test files or global setup:
  * **test.beforeAll()**: Setup that runs once before all tests in a file
  * **test.afterAll()**: Cleanup that runs once after all tests in a file
  * **test.beforeEach()**: Setup before each test (e.g., login, navigation)
  * **test.afterEach()**: Cleanup after each test (screenshots on failure are automatic with Playwright config)

**Example in steps file:**
```typescript
import { test, expect } from '@playwright/test';
import { LoginPage } from '../tests/pages/LoginPage';

test.describe('Product Browsing Tests', () => {
  test.beforeEach(async ({ page }) => {
    // Common setup - runs before each test
    const loginPage = new LoginPage(page);
    await loginPage.login(process.env.DS_USERNAME!, process.env.DS_PASSWORD!);
  });

  test('Verify product display', async ({ page }) => {
    // Test implementation
  });
});
```

**Browser Lifecycle Management**: Playwright automatically manages browser lifecycle. Screenshots, videos, and traces are captured automatically based on `playwright.config.ts` settings.

---

## 6️⃣-A Playwright Configuration File (`playwright.config.ts`)

Create a comprehensive `playwright.config.ts` with playwright-bdd configuration:

```typescript
import { defineConfig, devices } from '@playwright/test';
import { defineBddConfig, cucumberReporter } from 'playwright-bdd';
import * as dotenv from 'dotenv';

dotenv.config();

// Define BDD configuration - reads feature files and step definitions
const testDir = defineBddConfig({
  features: 'features/*.feature',
  steps: 'steps/*.ts',
});

export default defineConfig({
  testDir, // Generated from feature files
  
  // Execution settings
  fullyParallel: false,
  retries: process.env.CI ? 2 : 1,
  workers: 1,  // Single worker for OAuth apps
  
  // Reporters - Playwright + Cucumber
  reporter: [
    ['html', { outputFolder: 'playwright-report', open: 'never' }],
    ['json', { outputFile: 'reports/json/test-results.json' }],
    ['junit', { outputFile: 'reports/junit/results.xml' }],
    ['list'],
    cucumberReporter('json', { outputFile: 'reports/cucumber-report.json' }),
    cucumberReporter('html', { outputFile: 'reports/cucumber-report.html' }),
  ],
  
  // Test configuration
  use: {
    baseURL: process.env.baseUrl || 'https://ds-dev.marksandspencer.app/',
    trace: 'on',                    // Always capture traces
    screenshot: 'on',               // Capture screenshots
    video: 'on',                    // Record full video
    actionTimeout: 15000,           // 15s per action
    navigationTimeout: 30000,       // 30s per navigation
  },
  
  // Browser projects
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    // Uncomment for multi-browser testing:
    // {
    //   name: 'firefox',
    //   use: { ...devices['Desktop Firefox'] },
    // },
    // {
    //   name: 'webkit',
    //   use: { ...devices['Desktop Safari'] },
    // },
  ],
  
  // Timeouts
  timeout: 90000,                  // 90s per test
  globalTimeout: 90 * 60 * 1000,   // 90 minutes total
});
```

**Key Configuration Points:**
- `fullyParallel: false` - Sequential execution (required for OAuth apps)
- `retries` - Automatic retries on failure
- `workers: 1` - Single worker for OAuth apps (prevents session conflicts)
- `trace: 'on'` - Full execution traces for debugging
- `screenshot: 'on'` - Screenshots for each action
- `video: 'on'` - Full video recording
- `baseURL` - Base URL for all tests

### 🖥️ Viewport Configuration

**Always configure viewport explicitly** in `playwright.config.ts`:
```typescript
use: {
  viewport: { width: 1366, height: 768 },
  deviceScaleFactor: 1,  // Prevents zoom issues
}
```

| Issue | Solution |
|-------|----------|
| App appears "zoomed out" | Set `deviceScaleFactor: 1` |
| Elements not visible | Increase viewport width/height |
| Mobile view on desktop | Ensure viewport matches device |

### 🧭 Navigation Flow Discovery

**Never assume navigation paths - run discovery first:**
1. Use discovery tests to map navigation elements
2. Document the flow with explicit waits between steps
3. Use specific text selectors: `.selector:has-text("ExactText")`

**Key Pattern:**
```typescript
// ❌ WRONG: Might click wrong element
await page.locator('.home-bu-label').first().click();

// ✅ CORRECT: Explicit text selection + wait
await page.locator('.home-bu-label:has-text("LINGERIE")').click();
await page.waitForTimeout(2000); // Critical for Angular/React
```

### 🔍 Feature Detection

**Detect features before testing them:**
```typescript
// Skip test if feature doesn't exist
const hasFeature = await page.locator('.feature-element').count() > 0;
if (!hasFeature) {
  test.skip(true, 'Feature not available');
}
```

| Feature | Detection Method |
|---------|------------------|
| Hover Zoom | Check CSS `transform` property |
| Modal/Dialog | Look for overlay with high z-index |
| Drag & Drop | Verify `draggable` attribute |

---

## 7️⃣ Reporting and Artifacts

### Directory Structure
```
reports/
├── json/
│   └── playwright-report.json
├── junit/
│   └── junit.xml
└── playwright-report/           # Auto-generated HTML report
    └── index.html
```

**Note:** Reporters are configured in section 6️⃣-A `playwright.config.ts`. The configuration includes HTML, JSON, and JUnit reporters.

### Report Features
- **HTML Report**: Interactive report with traces, videos, and screenshots
  - View: `npx playwright show-report`
  - Location: `playwright-report/index.html`

- **JSON Report**: Machine-readable format for CI/CD integration
  - Location: `reports/json/playwright-report.json`

- **JUnit Report**: Standard XML format for CI systems (Jenkins, GitHub Actions)
  - Location: `reports/junit/junit.xml`

- **Artifacts Captured**:
  - ✅ Full execution traces (debugging)
  - ✅ Screenshots for each test action
  - ✅ Full video recording of test execution

### Package Scripts
Add to package.json:
```json
{
  "scripts": {
    "test": "npx playwright test",
    "test:headed": "npx playwright test --headed",
    "test:ui": "npx playwright test --ui",
    "test:debug": "npx playwright test --debug",
    "show-report": "npx playwright show-report"
  }
}
```

---

## 8️⃣ Naming Conventions

| Type     | Format           | Example                                   |
| -------- | ---------------- | ----------------------------------------- |
| Feature  | lowercase-hyphen | `login.feature`                           |
| Steps    | lowercase-hyphen | `login.steps.ts`                          |
| Page Obj | PascalCase       | `LoginPage.ts` or `login.page.ts`         |
| Scenario | Natural English  | `Successful login with valid credentials` |

**Consistency in Naming**: Ensure consistent naming in filenames, variables, and functions throughout the project. All TypeScript files use strict typing.

---

## 9️⃣ Common Issues and Solutions (Troubleshooting Guide)

### 🔄 AUTO-DISCOVERY PROTOCOL (When Tests Fail)

**When ANY test fails due to navigation, selector, or interaction issues, Copilot MUST:**

```
┌─────────────────────────────────────────────────────────────────┐
│  TEST FAILURE DETECTED                                          │
│  (TimeoutError, Element not found, Navigation failed, etc.)     │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  STEP 1: CREATE DISCOVERY TEST                                  │
│  - Create a @discovery test to inspect actual DOM               │
│  - Log all available selectors, classes, navigation elements    │
│  - Take screenshots at each step                                │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  STEP 2: RUN DISCOVERY TEST                                     │
│  npx playwright test --grep @discovery --headed                 │
│  - Observe actual application behavior                          │
│  - Extract real selectors from console output                   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  STEP 3: UPDATE CODE WITH DISCOVERED VALUES                     │
│  - Replace assumed selectors with actual ones                   │
│  - Update wait times based on observed load times               │
│  - Fix interaction patterns (click vs checkbox, etc.)           │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  STEP 4: RE-RUN ORIGINAL TEST                                   │
│  - Test should now pass with corrected values                   │
│  - If still failing, repeat from Step 1                         │
└─────────────────────────────────────────────────────────────────┘
```

**Quick Discovery Test Template:**
```typescript
test('Discover navigation/selectors @discovery', async ({ page }) => {
  // Login first
  await loginPage.login(process.env.DS_USERNAME!, process.env.DS_PASSWORD!);
  
  // Navigate to the problematic page
  console.log('Current URL:', page.url());
  
  // Extract all clickable elements
  const clickables = await page.evaluate(() => {
    return Array.from(document.querySelectorAll('a, button, [role="button"], [onclick]'))
      .slice(0, 20)
      .map(el => ({
        tag: el.tagName,
        text: el.textContent?.trim().substring(0, 30),
        class: el.className,
      }));
  });
  console.log('Clickable elements:', JSON.stringify(clickables, null, 2));
  
  // Extract all unique classes containing target keywords
  const classes = await page.evaluate(() => {
    const keywords = ['product', 'item', 'card', 'nav', 'menu', 'btn', 'button'];
    return Array.from(document.querySelectorAll('[class]'))
      .map(el => el.className)
      .filter(c => keywords.some(k => c.toLowerCase().includes(k)))
      .filter((v, i, a) => a.indexOf(v) === i);
  });
  console.log('Relevant classes:', classes);
  
  await page.screenshot({ path: 'reports/discovery-output.png' });
});
```

**DO NOT:**
- ❌ Retry the same failing code multiple times
- ❌ Add arbitrary waits without discovery
- ❌ Assume selectors based on common patterns
- ❌ Report "test failed" without investigating

**ALWAYS:**
- ✅ Create discovery test on first failure
- ✅ Extract actual DOM structure
- ✅ Update code with real values
- ✅ Validate fix before reporting

---

### 🐛 Issue: Tests Fail with "Element count = 0" Despite Successful Navigation

**Symptoms:**
- Navigation completes successfully (URL changes)
- `page.locator('.selector').count()` returns 0
- Screenshots show elements are visible

**Root Cause:**
- Assumed selectors don't match actual DOM
- Elements not loaded yet (Angular/React rendering delay)

**Solution:**
```typescript
// 1. Create debug test to discover actual selectors
test('Debug element discovery @debug', async ({ page }) => {
  await page.goto('/target-page');
  await page.waitForLoadState('networkidle');
  await page.waitForTimeout(2000); // Wait for Angular/React
  
  const domInfo = await page.evaluate(() => {
    return {
      allClasses: Array.from(new Set(
        Array.from(document.querySelectorAll('*'))
          .map(el => el.className)
          .filter(c => typeof c === 'string' && c.includes('product'))
      )),
      elementCount: document.querySelectorAll('[class*="product"]').length,
    };
  });
  
  console.log('Actual DOM classes:', domInfo);
});

// 2. Update page object with real selectors from DOM
this.productTiles = page.locator('.products-large'); // From actual DOM

// 3. Add wait logic in getProductCount method
async getProductCount(): Promise<number> {
  await this.page.waitForLoadState('networkidle');
  await this.page.waitForTimeout(2000);
  await this.productTiles.first().waitFor({ state: 'visible', timeout: 10000 });
  return await this.productTiles.count();
}
```

### 📋 Quick Troubleshooting Reference

| Issue | Symptom | Solution |
|-------|---------|----------|
| **Element count = 0** | Navigation works but locator returns 0 | Run discovery test, update selectors |
| **Zoomed out UI** | Elements appear tiny | Set `deviceScaleFactor: 1` in config |
| **Selector not found** | User confirms element exists | Discover actual text (case/spacing matters) |
| **Navigation timing** | Works in debug, fails in test | Add 2s waits between navigation steps |
| **Feature not detected** | Feature exists but test says no | Check correct element/CSS property |
| **Empty results** | Page loads but 0 products | Ask user for data-rich category path |
| **Wrong credentials** | Login fails with system username | Use `DS_USERNAME` prefix, not `username` |

### 📋 Troubleshooting Checklist

When tests fail, check in this order:

1. **Selector Issues** (90% of failures)
   - [ ] Created debug test to extract actual DOM selectors
   - [ ] Verified element count > 0 with discovered selectors
   - [ ] Updated page objects with real selectors

2. **Timing Issues** (5% of failures)
   - [ ] Added `waitForLoadState('networkidle')` after navigation
   - [ ] Added 2s timeout after navigation for Angular/React rendering
   - [ ] Used `waitFor({ state: 'visible' })` before interactions

3. **Test Data Issues** (3% of failures)
   - [ ] Verified selected category/department has data
   - [ ] Asked user for data-rich test paths
   - [ ] Confirmed products/items load in debug tests

4. **Configuration Issues** (2% of failures)
   - [ ] Set `viewport` and `deviceScaleFactor: 1` in config
   - [ ] Environment variables use `DS_` prefix (not `username`/`password`)
   - [ ] `.env` file exists and is loaded correctly

---

## 🔄 Iterative Development Workflow (Test-Fix-Validate)

**Philosophy: Execute tests IMMEDIATELY after generation, then fix failures iteratively.**

### Mandatory Post-Generation Workflow

**After generating any test artifacts, ALWAYS:**

1. **Execute All Tests Immediately**
   ```bash
   npm test
   ```

2. **Analyze Failures** (don't just report them - investigate):
   ```
   Example failure: "TimeoutError: locator.click: Timeout 30000ms exceeded"
   
   Investigation steps:
   - Check screenshot: Is element visible?
   - Check trace: What URL are we on?
   - Check selector: Does it exist in DOM?
   ```

3. **Create Debug Tests** for each failure type:
   ```typescript
   // Example: Product count = 0 failure
   test('Debug why product count is 0 @debug', async ({ page }) => {
     // Reproduce navigation
     // Extract actual DOM structure
     // Log findings
   });
   ```

4. **Fix Root Cause** (not symptoms):
   - ❌ BAD: Add arbitrary `waitForTimeout(5000)` everywhere
   - ✅ GOOD: Discover actual selector is `.products-large` not `.product-tile`

5. **Re-run Failed Test** (not entire suite):
   ```bash
   npx playwright test -g "specific failing test name"
   ```

6. **Validate Fix** - test should now PASS

7. **Move to Next Failure** - repeat steps 2-6

8. **Run Full Suite** when all individual tests pass:
   ```bash
   npm test
   ```

### Execution-First Mindset

**Traditional approach (WRONG):**
```
1. Generate all tests
2. Report to user: "Tests generated ✓"
3. User runs tests
4. User reports failures
5. You debug
```

**Correct approach (RIGHT):**
```
1. Generate tests
2. Execute tests immediately
3. See failures yourself
4. Debug and fix proactively
5. Re-run until passing
6. Report to user: "Tests generated and validated ✓ (6/6 passing)"
```

### Example Iterative Session

**Real example from this project:**

```
Iteration 1: Generate tests → Execute
Result: login.steps.ts 0/3 FAILED (selector issues)
Action: Detected OAuth, rewrote LoginPage with multi-step flow
Execute: login.steps.ts 3/3 PASSING ✓

Iteration 2: Execute product-browsing.steps.ts
Result: 0/6 FAILED (product count = 0)
Action: Created debug test, discovered `.products-large` selector
Execute: 1/6 PASSING

Iteration 3: Fix viewport
Result: Elements visible but still failing
Action: Set deviceScaleFactor: 1 in config
Execute: 1/6 PASSING

Iteration 4: Fix navigation
Result: User provided correct path (LINGERIE → T32)
Action: Updated navigation selectors with exact text
Execute: 4/6 PASSING, 2 SKIPPED

Iteration 5: Fix magnifier detection
Result: Feature exists but detection wrong
Action: Check CSS transform on ALL products, not just first
Execute: 6/6 PASSING ✓

Final: All tests validated and working
```

### Debug Test Patterns

**Keep debug tests for future reference:**

```typescript
// steps/debug/
// ├── selector-discovery.steps.ts    // DOM structure extraction
// ├── navigation-flow.steps.ts       // Navigation path discovery  
// ├── feature-detection.steps.ts     // Check if features exist
// └── test-data-validation.steps.ts  // Verify data loads

// Mark with @debug tag to exclude from main runs
test('Discover selectors @debug', async ({ page }) => {
  // ... discovery logic
});
```

**Playwright config:**
```typescript
// Run only non-debug tests by default
grep: /^(?!.*@debug)/,

// Run only debug tests when needed
// npx playwright test --grep @debug
```

### Success Criteria

**A test suite is "complete" when:**
- ✅ All tests execute without errors
- ✅ All assertions pass
- ✅ All selectors found from actual DOM (not assumed)
- ✅ Feature files align with spec files (scenario count matches test count)
- ✅ Navigation paths validated by user
- ✅ Test data produces results (not empty pages)
- ✅ Reports generated successfully
- ✅ No hardcoded waits without justification

**Documentation is complete when:**
- ✅ Test summary shows final results
- ✅ Known limitations documented (e.g., skipped tests with reasons)
- ✅ copilot-instructions.md updated with learnings
- ✅ User can re-run tests successfully

---

## 🔟 Assertions

Use Playwright's `expect()` API:

```typescript
import { expect } from '@playwright/test';

// URL assertions
await expect(page).toHaveURL(/.*inventory/);

// Element visibility
await expect(page.locator('#logout')).toBeVisible();

// Text content
await expect(page.locator('.error')).toContainText('Invalid credentials');
await expect(page.locator('.success')).toHaveText('Saved successfully');

// Element count
await expect(page.locator('.product-item')).toHaveCount(6);

// Attribute check
await expect(page.locator('.item')).toHaveAttribute('data-selected', 'true');

// CSS property check
await expect(page.locator('.zoom')).toHaveCSS('transform', /scale/);
```

**Code Quality Best Practices:**

* Clear selectors and consistent naming
* Prefer semantic locators (getByRole, getByText, getByPlaceholder) over CSS selectors
* No hard waits (`setTimeout`) — use Playwright waits exclusively
* Handle retries for dynamic elements
* Self-contained test execution
* Use TypeScript for full type safety
* All async operations must use `await`

---

## 1️⃣1️⃣ Test Execution & Validation Rules

### Post-Generation Validation Checklist

**After generating test files, ALWAYS validate:**

1. **Feature-Step Alignment**:
   - [ ] Count scenarios in each .feature file
   - [ ] Verify all Gherkin steps have definitions in corresponding .steps.ts file
   - [ ] Example: login.feature: 3 scenarios → login.steps.ts implements all Given/When/Then
   - [ ] Check that scenario titles are descriptive
   - [ ] Verify tags are consistent (@smoke, @regression, @P0)

2. **Authentication Flow Validation**:
   - [ ] Detect authentication type (OAuth vs Standard)
   - [ ] Implement correct LoginPage pattern
   - [ ] Test login flow manually first (headed mode: `npx playwright test --headed`)
   - [ ] Capture screenshots at each step to verify flow
   - [ ] Document actual flow in feature file Background section

3. **Environment Variables**:
   - [ ] Use prefixed names (DS_USERNAME, DS_PASSWORD) to avoid OS conflicts
   - [ ] Set credentials in PowerShell: `$env:DS_USERNAME='...'; $env:DS_PASSWORD='...'`
   - [ ] baseUrl goes in .env file, credentials do NOT
   - [ ] Test with: `console.log(process.env.DS_USERNAME)` in code
   - [ ] Ensure dotenv is configured in playwright.config.ts (for baseUrl only)

4. **Test Execution**:
   - [ ] Run tests in headed mode first: `npx playwright test --headed`
   - [ ] Fix failures iteratively (max 3 cycles)
   - [ ] Run in headless mode: `npm test`
   - [ ] Verify all reports generate (HTML, JSON, JUnit)
   - [ ] Create test-summary.txt with results

### Common Pitfalls & Solutions

| Issue | Symptom | Solution |
|-------|---------|----------|
| **Feature/Spec Mismatch** | Feature has 1 scenario but spec has 3 tests | Add missing scenarios to .feature file |
| **OAuth Not Detected** | Tests timeout on login | Check for URL redirects, implement OAuth flow |
| **Env Var Conflicts** | Username shows system user instead of test user | Rename to DS_USERNAME, DS_PASSWORD |
| **Stay Signed In Hangs** | Test stalls after password entry | Implement handleStaySignedIn() with try-catch |
| **Password Page Not Loading** | Password input not found after Next | Add 2+ second wait after clicking Next |
| **Tests Pass Locally, Fail in CI** | Timing issues in CI environment | Increase retries in playwright.config.ts for CI |

## 1️⃣2️⃣ Stabilization Rules

Before each test (configured in `playwright.config.ts`):

* Set `actionTimeout: 15000` in use config.
* Set `navigationTimeout: 30000` in use config.
* Use `waitUntil: 'networkidle'` after navigation.
* Enable trace capture: `trace: 'on'` for debugging.
* Enable screenshots: `screenshot: 'on'` for visual debugging.
* Enable videos: `video: 'on'` for full test recording.

---

## 🗣️ Prompt Schema (How to Interpret User Prompts)

When the user provides a natural-language request such as:

> “Generate Playwright + CucumberJS UI test scripts for verifying the Login page at `https://practicetestautomation.com/practice-test-login/` with positive and negative scenarios.”

You must:

### 🧩 1. Parse the Intent

Extract from the prompt:

* Website URL(s)
* User actions (click, enter text, navigate)
* Expected outcomes or validations
* Scenario names (positive/negative/etc.)

### 2. Generate Test Files

Generate:

* **Feature file** in `features/` - BDD scenarios (Gherkin).
* **Step definitions** in `steps/` - Implements Given/When/Then steps.
* **Page Object** (if applicable) under `tests/pages/`.

### 🔁 3. Apply Self-Healing and Retry Logic

Automatically detect unstable selectors or navigation failures. Use auto-retry wrappers and fallback queries.

### 🔄 4. Self-Correction Cycle

If a test fails, analyze the error, regenerate the failing part, and retry until it passes.

### ⚡ 5. Generation Behavior

Automatically fix syntax/import issues. Output runnable, self-healing tests.

---

## 📊 Requirements-to-Tests Workflow

### Overview
This project uses a **structured requirements file** (`requirements.xlsx`) and environment configuration as inputs for test automation.

### Input Files

> **📌 See Section 2️⃣-A for detailed environment configuration:**
> - `.env` file contains `baseUrl` only (NO credentials)
> - Credentials via PowerShell: `$env:DS_USERNAME='...'; $env:DS_PASSWORD='...'`

**`requirements.xlsx` (Test Scenarios):**
- Business Analysts/QA maintain test flows
- Contains user journeys and acceptance criteria

### Complete Flow:
```
┌─────────────────────┐       ┌─────────────────────┐
│        .env         │       │  requirements.xlsx  │
│  (baseUrl only)     │       │  (Test Scenarios)   │
└──────────┬──────────┘       └──────────┬──────────┘
           │                             │
           └──────────────┬──────────────┘
                          │
                          ▼
               ┌─────────────────────┐
               │   user-journeys.md  │  ← Auto-generated
               └──────────┬──────────┘
                          │
                          ▼
               ┌─────────────────────┐
               │  features/*.feature │  ← BDD scenarios (Gherkin)
               └──────────┬──────────┘
                          │
                          ▼
               ┌─────────────────────┐
               │  steps/*.steps.ts   │  ← Step definitions
               └──────────┬──────────┘
                          │
                          ▼
               ┌─────────────────────┐
               │  tests/pages/*.ts   │  ← Page objects
               └─────────────────────┘
```

Note: Credentials (DS_USERNAME, DS_PASSWORD) are set via PowerShell environment variables, not in .env.

### Workflow Steps

#### Step 1: Understanding Requirements Format
Business Analysts and QA maintain manual test flows in `requirements.xlsx` in a **free-form acceptance criteria format**. The Excel may contain:

**Typical Format:**
- **Column A**: Journey/Step number (e.g., `1`, `2`, `3` or blank)
- **Column B**: Feature/Action name (e.g., `Range Creation`, `Select Products`, `Launch Range Builder`)
- **Column C**: Detailed description of the step or acceptance criteria
- **Multiple sheets**: Different sheets may contain different test scenarios or journey descriptions

**Example (actual format from requirements.xlsx):**
```
Sheet1:
  | Regression Testcases | Login -> Home Page -> BU -> Dept -> PLP Page
1 | Range Creation       |
1 | Select Products      | Choose products from the Product Listing Page (PLP).
2 | Launch Range Builder | Move selected products into the Range Builder tool.
3 | Review Range Basket (Left Side) | The chosen products will appear on the left side, referred to as the Range basket.
4 | Populate the Range (Right Side) | Drag and drop products from the basket to the right side, which is the Range Window.
5 | Name the Range       | Enter a name for the new Range.
6 | Save Operation       | Upon clicking Save, the Range is created and stored in the DS database.
7 | Publish the Range    | Select 'Publish' by right click to finalize and publish the Range

2 | Product Browsing     |
1 | Product Tile         | Product tile contains information such as New Flag, Product Description, Color, Price, Department...

Sheet2:
Login Journey: Navigate to baseUrl → Enter username (PowerShell env) → Enter password (PowerShell env) → Click Login → Verify Home Page
```

**Key Characteristics:**
- No strict schema - BA/QA write in natural language
- **Journey grouping**: New journey starts when Column A has a new number and Column C is empty (e.g., `1 | Range Creation |`)
- **Sequential step numbering**: Each step under a journey is numbered 1, 2, 3, 4, etc.
- Descriptions are in plain English acceptance criteria format
- May include flow arrows (→) and page transitions
- References to .env variables may appear in descriptions (e.g., "from .env")

---

#### Step 2: Intelligent Parsing & Journey Generation
When prompted with **"Generate UI test"** or **"Generate user journeys from requirements"**, Copilot should:

1. **Read all sheets in `requirements.xlsx`**
2. **Parse free-form content intelligently**
3. **Extract structured journeys** using NLP and pattern recognition
4. **Create `user-journeys.md`** with standardized format

**Intelligent Parsing Rules:**

**1. Identify Test Scenarios/Features:**
- Look for journey headers: rows where Column A has a number, Column B has feature name, and Column C is empty
- Pattern: `{number} | {Feature Name} | ` (empty description indicates journey header)
- Examples: `1 | Range Creation |`, `2 | Product Browsing |`
- Use feature name from Column B as the journey name

**2. Extract Test Steps:**
- Steps follow immediately after the journey header
- Each step has: Column A = step number (1, 2, 3...), Column B = action name, Column C = description
- Sequential numbering (1, 2, 3, 4, 5...) within each journey
- Reset to 1 when a new journey starts

**3. Detect Journey Boundaries:**
- New journey starts when: Column A changes to a new number AND Column C is empty
- All steps between two journey headers belong to the first journey
- Continue reading until next journey header or end of sheet

**4. Infer Page Context:**
- Extract page names from descriptions (e.g., "Product Listing Page (PLP)" → Page: `PLP`)
- Look for keywords: "page", "screen", "modal", "dialog", "window"
- Common pages: `LoginPage`, `HomePage`, `PLP`, `RangeBuilder`

**5. Determine Actions:**
- Parse action verbs from descriptions:
  - "Choose", "Select" → `click` or `select`
  - "Enter", "Type", "Fill" → `fill`
  - "Click", "Press" → `click`
  - "Drag and drop" → `dragDrop`
  - "Right click" → `rightClick`
  - "Navigate", "Go to" → `navigate`
  - "Hover" → `hover`
  - "Move" → could be `click` or `dragDrop` (use context)

**6. Extract Expected Outcomes:**
- Look for result-oriented language:
  - "will appear", "is displayed", "shows" → Assertion: `visible`
  - "contains", "shows text" → Assertion: `text`
  - "redirected to", "navigates to" → Assertion: `url`
  - "count", "number of" → Assertion: `count`
  - "saved", "stored", "created" → Assertion: `text` (look for success message)

**7. Identify Test Data:**
- Look for credential references (e.g., "username" → `${DS_USERNAME}` from PowerShell env)
- Look for baseUrl references (e.g., "navigate to site" → `${baseUrl}` from .env)
- Quoted values or specific data (e.g., "Fashion" → `"Fashion"`)
- Generic descriptions (e.g., "products" → use placeholder or test data)

**8. Detect Preconditions:**
- Look for header rows or phrases like "Login ->", "User is logged in"
- Top of sheet descriptions often indicate prerequisites
- Flow arrows (→) may show prerequisite steps

**9. Handle Flow Descriptions:**
- Parse arrow-based flows (e.g., "Login → Home → BU → Dept")
- Convert each segment into a journey step
- Example: "Navigate to baseUrl → Enter username" becomes two steps

**10. Journey Metadata Inference:**
- **Module**: Use top-level feature name (e.g., "Range Creation" → Module: `RangeBuilder`)
- **Tags**: Default to `@regression`; add `@smoke` for critical paths
- **Priority**: Default to `P1`; set `P0` for main happy paths
- **Preconditions**: Extract from flow headers or first step context

---

**Output Format for `user-journeys.md`:**

Create structured journeys from the free-form Excel content using this template:

```markdown
# User Journeys
*Auto-generated from requirements.xlsx*

## Journey: [Feature Name] - Happy Path (ID: JRNY-[number])
**Module**: [ModuleName inferred from feature]  
**Tags**: @regression @smoke  
**Priority**: P0  
**Preconditions**: [Extracted from flow header or context]  
**Start**: [First page/action] → **End**: [Last page/state]

### Steps:
1. **[PageName]** - [action]: [target element description]
   - Data: [test data or ${envVariable}]
   - Expected: [expected outcome from description]
   - Assert: [assertionType]
   - Notes: [additional context if needed]
```

**Example Transformation (Range Creation from actual Excel):**

Excel Input:
```
1 | Range Creation       |
1 | Select Products      | Choose products from the Product Listing Page (PLP).
2 | Launch Range Builder | Move selected products into the Range Builder tool.
3 | Review Range Basket (Left Side) | The chosen products will appear on the left side, referred to as the Range basket.
4 | Populate the Range (Right Side) | Drag and drop products from the basket to the right side, which is the Range Window.
5 | Name the Range       | Enter a name for the new Range.
6 | Save Operation       | Upon clicking Save, the Range is created and stored in the DS database.
7 | Publish the Range    | Select 'Publish' by right click to finalize and publish the Range
```

Generated Journey Output:
```markdown
## Journey: Range Creation - Happy Path (ID: JRNY-001)
**Module**: RangeBuilder  
**Tags**: @regression @smoke  
**Priority**: P0  
**Preconditions**: User is logged in and navigated to PLP (Login -> Home -> BU -> Dept -> PLP)  
**Start**: PLP → **End**: Range Published

### Steps:
1. **PLP** - click: Product checkbox/selector
   - Data: Multiple product IDs
   - Expected: Choose products from the Product Listing Page (PLP)
   - Assert: visible
   - Notes: Select products from PLP

2. **PLP** - click: "Launch Range Builder" button
   - Expected: Range Builder tool opens, selected products are moved into it
   - Assert: visible

3. **RangeBuilder** - verify: Range Basket (left side)
   - Expected: The chosen products will appear on the left side, referred to as the Range basket
   - Assert: visible
   - Notes: Review Range Basket on left side

4. **RangeBuilder** - dragDrop: Products from basket to Range Window (right side)
   - Expected: Products appear on the right side, which is the Range Window
   - Assert: visible
   - Notes: Drag and drop products from basket to Range Window

5. **RangeBuilder** - fill: Range name input field
   - Data: "Test Range - " + timestamp
   - Expected: Enter a name for the new Range
   - Assert: value

6. **RangeBuilder** - click: Save button
   - Expected: Upon clicking Save, the Range is created and stored in the DS database
   - Assert: text
   - Notes: Verify save operation and database storage confirmation

7. **RangeBuilder** - rightClick: Saved Range item in list
   - Expected: Context menu appears
   - Assert: visible

8. **RangeBuilder** - click: "Publish" option from context menu
   - Expected: Range is finalized and published
   - Assert: text
   - Notes: Select 'Publish' by right click to finalize and publish the Range
```

**Note:** Generated journey may have more steps than Excel if compound actions are split (e.g., "right click and publish" becomes two steps: rightClick + click).

---

**Parsing Strategy:**

1. **Detect journey headers**: Look for pattern `{number} | {Feature Name} | ` (where Column C is empty)
2. **Extract feature name** from Column B as the journey name
3. **Collect all subsequent steps**: Read rows with step numbers (1, 2, 3...) until next journey header
4. **Parse each step row**:
   - Column A = step sequence number
   - Column B = action/step name
   - Column C = detailed acceptance criteria description
5. **Analyze description text** to determine:
   - Page context (look for "page", specific page names like "PLP", "Range Builder")
   - Action type (verb-based: choose, move, enter, click, drag, etc.)
   - Expected outcomes (will appear, shows, displays, etc.)
   - Test data references (.env variables, specific values)
6. **Auto-generate Journey IDs** sequentially (JRNY-001, JRNY-002, etc.)
7. **Infer page names** from context or create standardized names (LoginPage, HomePage, PLP, RangeBuilder)

---

**Quality Checks for Generated Journeys:**
- Ensure every journey has at least one step
- Verify steps are logically sequenced
- Confirm all `${envVariable}` references match variables in `.env` file
- Check that `Page` names are valid PascalCase identifiers for page object generation
- Validate inferred actions match supported Playwright actions
- Add journey separators (`---`) between different test scenarios

**Error Handling:**
- If `requirements.xlsx` is missing or unreadable: Prompt user to check file
- If Excel is completely empty: Provide guidance on filling acceptance criteria
- If unable to parse steps: Use best-effort inference and add notes for manual review
- If .env references don't exist: Flag warnings but continue generation

---

#### Step 3: Generate Playwright Artifacts from User Journeys
When prompted with **"Generate Playwright tests from user journeys"**, Copilot should:

1. Read `user-journeys.md`
2. For each journey, generate:
   - **Feature file** (`features/[module-name].feature`) with Gherkin syntax - EXECUTED via playwright-bdd
   - **Step definitions** (`steps/[module-name].steps.ts`) - Implements Given/When/Then steps
   - **Page objects** (`tests/pages/[PageName]Page.ts`) - e.g., `LoginPage.ts`, `PLPPage.ts`

**Generation Rules:**
- **Feature file**: Convert journey steps into Given/When/Then format (executed via playwright-bdd)
- **Step definitions**: Implement each Gherkin step with Playwright actions
- **Page objects**: Create methods for each unique page action (click, fill, select, dragDrop)
- **Test data**: Use `${DS_USERNAME}` (PowerShell env), `${baseUrl}` (from .env)
- **Assertions**: Use appropriate Playwright expectations based on AssertionType:
  - `visible` → `await expect(locator).toBeVisible()`
  - `text` → `await expect(locator).toContainText(expectedText)`
  - `url` → `await expect(page).toHaveURL(expectedUrl)`
  - `count` → `await expect(locator).toHaveCount(expectedCount)`
  - `value` → `await expect(locator).toHaveValue(expectedValue)`

**Naming conventions from journey data:**
- Feature file: `features/[Module-lowercase-hyphen].feature`
- Step file: `steps/[Module-lowercase-hyphen].steps.ts`
- Page objects: `tests/pages/[Page]Page.ts` (e.g., `HomePage.ts`, `PLPPage.ts`, `RangeBuilderPage.ts`)

---

#### Step 4: Validation and Execution (MANDATORY)
**This step is REQUIRED - do not skip:**

1. **Validate file creation**: Confirm all artifacts are generated
2. **Execute tests**: Run `npm test` immediately after generation
3. **Monitor results**: Check for failures and errors
4. **Fix failures**: If any test fails:
   - Debug the root cause (selector issues, timing, navigation, assertions)
   - Apply fixes to page objects, specs, or configuration
   - Re-run failed tests
   - Iterate until all tests pass
5. **Generate reports**: 
   - HTML report: `npx playwright show-report`
   - Create `test-summary.txt` with execution results
6. **Inform user**: Report final status (all passing or specific issues)

**Do not consider the task complete until tests are executed and validated.**

---

### Usage Instructions

**Primary Workflow - Generate Everything from Requirements:**
When user prompts with **"Generate UI test"**, Copilot should:
1. Read `requirements.xlsx` and parse all sheets intelligently
2. **Auto-generate `user-journeys.md`** (do not ask user to create this manually)
3. **Auto-generate Playwright test artifacts** (features, specs, page objects)
4. **Execute tests immediately** (`npm test`)
5. **Fix any failures** and re-run until all tests pass
6. **Generate reports** and provide execution summary to user

**Execution is mandatory - do not skip step 4-6.**

**Alternative Workflows:**

**To only generate user journeys from Excel:**
```
Prompt: "Generate user journeys from requirements.xlsx"
```
*Result*: Creates `user-journeys.md` only (no test files)

**To generate Playwright tests from existing journeys:**
```
Prompt: "Generate Playwright tests from user-journeys.md"
```
*Result*: Reads existing `user-journeys.md` and creates test artifacts

**To do both in one step:**
```
Prompt: "Read requirements.xlsx, generate user journeys, and create Playwright test automation"
```
*Result*: Full pipeline execution

---

**Important Notes:**
- **`user-journeys.md` is auto-generated** - Do NOT manually create or commit this file initially
- **`requirements.xlsx` is the source of truth** - Manually maintained by BA/QA teams
- **Regeneration**: If requirements change, re-run "Generate UI test" to update journeys and tests
- **Version Control**: Consider `.gitignore` for `user-journeys.md` if it's purely intermediate, OR commit it for review/traceability

---

## 🚨 Common Pitfalls and Anti-Patterns (AVOID THESE)

### Anti-Pattern Summary Table

| Anti-Pattern | Problem | Solution |
|-------------|---------|----------|
| Generic selectors | `.product-card` doesn't exist | Run discovery, use actual selectors |
| Assumed checkboxes | App uses click selection | Discover selection mechanism first |
| Playwright dragTo | Custom drag implementation | Use native mouse events |
| Expected save button | App uses icon + Enter | Discover save mechanism first |
| Expected toast/modal | App has no confirmation UI | Check state instead |
| password.fill() | Special chars fail | Use evaluate() method |
| toContain(regex) | Wrong assertion syntax | Use toMatch(regex) |
| Multiple workers | OAuth conflicts | Use workers: 1 |
| No wait after nav | Elements not loaded | Wait 6+ seconds for Angular |

**Key Code Patterns:**

```typescript
// Password with special characters - use evaluate()
await passwordInput.evaluate((el: HTMLInputElement, pwd: string) => {
  el.value = pwd;
  el.dispatchEvent(new Event('input', { bubbles: true }));
  el.dispatchEvent(new Event('change', { bubbles: true }));
}, password);

// Native mouse drag (when dragTo fails)
await page.mouse.move(sourceX, sourceY);
await page.mouse.down();
await page.mouse.move(targetX, targetY, { steps: 10 });
await page.mouse.up();

// Regex assertion - use toMatch, not toContain
expect(message).toMatch(/saved|success/i);
```

---

## 📋 Pre-Generation Checklist

**Before generating ANY test code, confirm:**

- [ ] **Phase 1 Complete**: Authentication flow discovered and documented
- [ ] **Phase 2 Complete**: Navigation patterns and wait times documented
- [ ] **Phase 3 Complete**: Actual selectors extracted from DOM (not assumed)
- [ ] **Phase 4 Complete**: Interaction patterns discovered (click/checkbox/keyboard)
- [ ] **Phase 5 Complete**: Drag-drop mechanism tested (if applicable)
- [ ] **Password Handling**: Special characters tested, evaluate() method used if needed
- [ ] **Wait Times**: Angular/React load times measured and documented
- [ ] **Save/Submit**: Actual mechanism discovered (button/icon/Enter/keyboard)
- [ ] **Confirmation**: Actual feedback mechanism documented (or "none")

**Only after all discovery phases pass should test generation begin.**

---