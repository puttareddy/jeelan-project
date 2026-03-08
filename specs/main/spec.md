# Modernization Specification

## Overview

Single-Page-Express is a client-side implementation of the Express.js route API that enables isomorphic routing between server and browser. The library works by hijacking link clicks and form submissions, then providing Express-compatible request and response objects to handle "requests" (click or submit events) and issue "responses" in the form of DOM updates.

This modernization project aims to replace the Node.js/Express backend with Python/FastAPI while preserving all existing frontend functionality. The goal is to enable isomorphic routing with a Python backend while maintaining 100% Express-compatible API surface for frontend developers. The library currently supports Express 4 and 5 routing syntax, Teddy templating engine, and includes advanced features like View Transitions API integration, scroll position memory, focus management, and accessibility support.

## Current Architecture

### File Structure

```
/Users/puttaiaharugunta/codebase/infi/single-page-express/
├── single-page-express.js          # Core library (786 lines)
├── webpack.config.js                # Multi-format build configuration
├── package.json                     # Dependencies and scripts
├── README.md                        # Project overview
├── USAGE.md                         # Usage documentation
├── CONFIGURATION.md                 # Configuration documentation
├── test/
│   └── tests.spec.js                # Playwright E2E tests
└── sampleApps/
    ├── basicFrontendOnly/
    │   ├── index.html
    │   └── index.js
    ├── basicFrontendOnlyWithTemplating/
    │   ├── index.html
    │   └── index.js
    ├── express/
    │   ├── server.js                  # Express server entry point
    │   ├── browser.js                 # Frontend entry point
    │   ├── package.json
    │   ├── styles.css
    │   └── mvc/
    │       ├── routes.js                # Shared route definitions
    │       ├── models/
    │       │   ├── browser/
    │       │   │   └── getRandomNumber.js
    │       │   └── server/
    │       │       └── getRandomNumber.js
    │       └── views/
    │           ├── index.html
    │           ├── layout.html
    │           ├── pageWithDataRetrieval.html
    │           ├── pageWithForm.html
    │           └── secondPage.html
    └── express-complex/
        ├── server.js
        ├── browser.js
        ├── package.json
        ├── styles.css
        └── mvc/
            ├── routes.js
            ├── models/
            └── views/
                ├── index.html
                ├── layout.html
                ├── params.html
                ├── pageWithDataRetrieval.html
                ├── pageWithForm.html
                ├── partial.html
                ├── secondPage.html
                └── verbosePage.html
```

### Frontend Architecture

**Core Module:** `single-page-express.js` (786 lines)

The core library implements an Express-like routing system in the browser:

- **Route Handler:** Hijacks `<a>` tag clicks and `<form>` submissions
- **Request Object:** Partial Express `req` implementation adapted for browser context with properties:
  - `req.app`, `req.baseUrl`, `req.body`, `req.cookies`
  - `req.hostname`, `req.ip`, `req.ips`, `req.method`
  - `req.originalUrl`, `req.params`, `req.path`, `req.protocol`, `req.query`
  - `req.route`, `req.secure`, `req.signedCookies`, `req.subdomains`, `req.xhr`
  - SPE-specific: `req.singlePageExpress`, `req.backButtonPressed`, `req.forwardButtonPressed`

- **Response Object:** Express-compatible `res` implementation:
  - Methods: `res.cookie()`, `res.clearCookie()`, `res.redirect()`, `res.render()`, `res.json()`
  - SPE-specific properties: `res.title`, `res.target`, `res.focus`, `res.beforeRender`, `res.afterRender`
  - Head tag management: `res.removeMetaTags`, `res.removeStyleTags`, `res.removeLinkTags`, `res.removeScriptTags`, `res.removeBaseTags`, `res.removeTemplateTags`, `res.removeHeadTags`
  - View transition control: `res.skipViewTransition`, `res.updateDelay`, `res.resetScroll`

- **Default Render Method:** Intelligent DOM manipulation with:
  - Targeted element replacement via `res.target` or `app.defaultTarget`
  - Automatic title updates from `res.title` or `<title>` tags
  - Head tag management (adding/updating `<link>`, `<script>`, `<meta>`, `<style>` tags)
  - View Transitions API integration with CSS transition fallback
  - Scroll position memory and restoration for page and child containers
  - Focus management for accessibility
  - Screen reader announcements via ARIA live regions
  - Pre/post-render hooks: `app.beforeEveryRender()`, `app.afterEveryRender()`, `res.beforeRender()`, `res.afterRender()`
  - Post-render callbacks: `app.postRenderCallbacks`

- **Event Listeners:**
  - Click events on anchor tags
  - Submit events on forms
  - Popstate events for back/forward button handling

- **Loading Indicator:** Optional Topbar integration (https://buunguyen.github.io/topbar/)

### Backend Architecture (Current - Express)

**Express Server Pattern:**

- MVC structure in sample apps (`sampleApps/express/`, `sampleApps/express-complex/`)
- Shared routes between server and frontend via `routes.js` module
- Template bundling: Templates are bundled into `public/templates.js` for client-side delivery
- Teddy templating engine integration
- Webpack bundling for frontend JavaScript
- Static file serving from `public/` directory
- Body parsing for form submissions
- JSON API endpoints (e.g., `/api/randomNumber`)

**Key Components:**

- `single-page-express.js` - Core library implementing Express API on client
- `webpack.config.js` - Multi-format build configuration (ESM, CommonJS, UMD, minified variants)
- Sample apps demonstrating:
  1. Basic frontend-only (no backend)
  2. Frontend with client-side templating
  3. Express integration with shared routes/templates
  4. Express complex integration with advanced features

### Build and Distribution

- **Webpack Configuration:** Produces 5 bundle formats:
  - `dist/single-page-express.mjs` - ES module (development)
  - `dist/single-page-express.cjs` - CommonJS/UMD bundle (development)
  - `dist/single-page-express.js` - Standalone UMD bundle (development)
  - `dist/single-page-express.min.mjs` - Minified ES module (production)
  - `dist/single-page-express.min.js` - Minified standalone UMD bundle (production)

- **Package Exports:**
  ```json
  "exports": {
    ".": {
      "import": "./dist/single-page-express.mjs",
      "require": "./dist/single-page-express.cjs"
    },
    "./min": {
      "default": "./dist/single-page-express.min.mjs"
    }
  }
  ```

- **npm Package:** Published as `single-page-express` version 2.0.5

## Target Architecture

### Backend Migration to FastAPI

**New Backend Structure:**

```
/Users/puttaiaharugunta/codebase/infi/single-page-express/
└── sampleApps/
    ├── fastapi/                      # New FastAPI sample (replaces express/)
    │   ├── server.py                  # FastAPI server entry point
    │   ├── browser.js                 # Frontend entry point (unchanged from express)
    │   ├── pyproject.toml              # Python dependencies
    │   ├── styles.css
    │   └── mvc/
    │       ├── routes.js                # Shared route definitions (unchanged)
    │       ├── routes.yaml              # New: Route configuration for FastAPI
    │       ├── models/
    │       │   ├── browser/
    │       │   │   └── getRandomNumber.js
    │       │   └── server/
    │       │       └── getRandomNumber.py  # New: Python model equivalent
    │       └── views/
    │           ├── index.html           # Unchanged (Teddy templates)
    │           ├── layout.html
    │           ├── pageWithDataRetrieval.html
    │           ├── pageWithForm.html
    │           └── secondPage.html
    └── fastapi-complex/              # New FastAPI complex sample (replaces express-complex/)
        ├── server.py
        ├── browser.js
        ├── pyproject.toml
        ├── styles.css
        └── mvc/
            ├── routes.js
            ├── routes.yaml
            ├── models/
            └── views/
```

**FastAPI Server Implementation:**

- Python 3.8+ with FastAPI (latest stable)
- Uvicorn ASGI server with hot reload support
- Route registration via YAML configuration or JavaScript route bridge
- Teddy template rendering via JavaScript engine integration (PyExecJS or subprocess)
- Static file serving for bundled frontend JavaScript
- Form data parsing with `python-multipart`
- JSON API endpoints with FastAPI's automatic OpenAPI documentation
- Jinja2 as primary templating option (optional, Teddy as primary for isomorphism)

### Route Definition Strategy

**Shared Route Format (YAML Configuration):**

```yaml
# sampleApps/fastapi/mvc/routes.yaml
routes:
  - method: GET
    path: /
    template: index
    handler: renderIndex

  - method: GET
    path: /secondPage
    template: secondPage
    model:
      variable: 'variable with contents: "hi there!"'

  - method: POST
    path: /pageWithForm
    template: pageWithForm
    parseBody: true

  - method: GET
    path: /pageWithDataRetrieval
    template: pageWithDataRetrieval
    async: true
    model: getRandomNumber

  - method: GET
    path: /route/:withParam/:anotherParam
    template: params
    model:
      withParam: req.params.withParam
      anotherParam: req.params.anotherParam
```

**JavaScript Route Bridge (Alternative):**

Keep existing `routes.js` format for backward compatibility and use a Python bridge to interpret it:

```python
# FastAPI bridge for JavaScript routes
import subprocess
import json

def load_js_routes(routes_path):
    """Load routes from JavaScript file via Node.js"""
    result = subprocess.run(
        ['node', '-e', f'module.exports = require("{routes_path}")'],
        capture_output=True,
        text=True
    )
    return json.loads(result.stdout)
```

### Template Sharing Strategy

**Option 1: Keep Teddy with Python Integration (Recommended)**

- Continue using Teddy templates on both sides
- Python integrates Teddy via:
  - PyExecJS: Execute JavaScript template engine in Python
  - Subprocess: Spawn Node.js process for template rendering
  - JavaScript engine: V8 wrapper in Python

**Option 2: Jinja2 with Client-Side Jinja2**

- Use Jinja2 templates on Python backend
- Use `jinja-js` or similar on JavaScript frontend
- Pros: Standard Python stack, good tooling
- Cons: Subtle differences between server and client implementations

**Option 3: Template Compilation**

- Pre-compile templates to a shared intermediate format
- Both Python and JavaScript load and execute compiled templates
- Pros: Best of both worlds
- Cons: Complex to implement

**MVP Recommendation:** Start with Option 1 (Keep Teddy) to minimize changes and ensure compatibility.

### Development Workflow

**Separate Servers:**

```bash
# Terminal 1: Backend (FastAPI)
cd sampleApps/fastapi
uvicorn server:app --reload --port 3000

# Terminal 2: Frontend (if separate dev server needed)
npm run build  # Build single-page-express
npm run sample-app-basic-frontend-only  # Run frontend-only sample
```

**Testing Workflow:**

```bash
# Frontend tests (unchanged)
npm test

# Backend tests (new)
pytest sampleApps/fastapi/tests/

# Integration tests (updated for FastAPI)
npm run test-integration
```

## Data Models

No persistent database models. Data is passed through:

### Request Object Properties

- **req.params**: Route parameters (e.g., `/users/:id` → `{id: "123"}`)
- **req.query**: Query string parameters (e.g., `?page=2` → `{page: "2"}`)
- **req.body**: Form submission data or JSON body
- **req.cookies**: Parsed cookies from `document.cookie`
- **req.path**: Current URL pathname
- **req.method**: HTTP method (GET, POST, etc.)
- **req.url**: Full URL including query string
- **req.app**: Reference to app instance
- **req.hostname**: Current hostname
- **req.originalUrl**: Original URL before Express routing
- **req.route**: Matched route information
- **req.secure**: Boolean indicating HTTPS
- **req.subdomains**: Array of subdomains
- **req.ip**: Client IP (stubbed as `127.0.0.1`)
- **req.ips**: Array of IPs (stubbed as empty)
- **req.baseUrl**: Base URL (stubbed as empty string)
- **req.signedCookies**: Signed cookies (stubbed as empty object - not supported in browser)
- **req.fresh**: Boolean (stubbed as `true`)
- **req.stale**: Boolean (stubbed as `false`)
- **req.xhr**: Boolean indicating XHR (stubbed as `true`)
- **req.singlePageExpress**: Boolean indicating SPE context (SPE-specific)
- **req.backButtonPressed**: Boolean (SPE-specific)
- **req.forwardButtonPressed**: Boolean (SPE-specific)

### Response Object Properties

- **res.title**: Override page title
- **res.target**: CSS selector for DOM update target
- **res.focus**: CSS selector for focus element after render
- **res.beforeRender**: Function to execute before DOM update
- **res.afterRender**: Function to execute after DOM update
- **res.removeMetaTags**: Boolean to remove all meta tags from head
- **res.removeStyleTags**: Boolean to remove all style tags from head
- **res.removeLinkTags**: Boolean to remove all link tags from head
- **res.removeScriptTags**: Boolean to remove all script tags from head
- **res.removeBaseTags**: Boolean to remove all base tags from head
- **res.removeTemplateTags**: Boolean to remove all template tags from head
- **res.removeHeadTags**: Boolean to remove all non-title head tags
- **res.skipViewTransition**: Boolean to skip view transition animation
- **res.updateDelay**: Number of milliseconds to delay DOM update
- **res.resetScroll**: Boolean to reset scroll position
- **res.app**: Reference to app instance
- **res.headersSent**: Boolean (stubbed)
- **res.locals**: Object for local variables (stubbed)

### Model Data Pattern

Models are simple JavaScript functions returning data objects:

```javascript
// sampleApps/express/mvc/models/server/getRandomNumber.js
module.exports = async function () {
  return {
    randomNumber: Math.floor(Math.random() * 10) + 1
  }
}
```

Equivalent Python version:

```python
# sampleApps/fastapi/mvc/models/server/getRandomNumber.py
import random

async def get_random_number():
    return {
        "randomNumber": random.randint(1, 10)
    }
```

## API Endpoints

### Frontend API (Express-Compatible - Preserved)

**Route Definition Methods:**

- `app.get(middleware?, route, callback)` - Register GET route
- `app.post(middleware?, route, callback)` - Register POST route
- `app.put(middleware?, route, callback)` - Register PUT route
- `app.delete(middleware?, route, callback)` - Register DELETE route
- `app.all(middleware?, route, callback)` - Register route for all methods
- `app.route(path)` - Chainable route registration
  - `.get(middleware?, callback)` - Register GET handler
  - `.post(middleware?, callback)` - Register POST handler
  - `.put(middleware?, callback)` - Register PUT handler
  - `.delete(middleware?, callback)` - Register DELETE handler
  - Supports all HTTP verbs: checkout, copy, delete, get, head, lock, merge, mkactivity, mkcol, move, m-search, notify, options, patch, post, purge, put, report, search, subscribe, trace, unlock, unsubscribe

**Configuration Methods:**

- `app.set(name, value)` - Set app configuration
- `app.get(name)` - Get app configuration
- `app.enable(name)` - Enable setting
- `app.disable(name)` - Disable setting
- `app.enabled(name)` - Check if setting is enabled
- `app.disabled(name)` - Check if setting is disabled

**App Settings:**

- `case sensitive routing` - Enable case-sensitive routing (default: false)
- `env` - Environment (default: "production")
- `query parser` - Enable query string parsing (default: true)
- `strict routing` - Enable strict trailing slash handling (default: false)
- `subdomain offset` - Subdomain offset (default: 2)

**Programmatic Methods:**

- `app.triggerRoute(params)` - Trigger route programmatically
  - `route`: Route to trigger
  - `method`: HTTP method (default: GET)
  - `body`: Request body data
  - `event`: Event object (for form submissions)
  - `parseBody`: Boolean to parse form data
  - `skipHistory`: Boolean to skip history update

**Request Methods (Express-Compatible):**

- `req.accepts(types)` - Check accepted content types
- `req.acceptsCharsets(charsets)` - Check accepted charsets
- `req.acceptsEncodings(encodings)` - Check accepted encodings
- `req.acceptsLanguages(langs)` - Check accepted languages
- `req.get(field)` - Get request header
- `req.is(type)` - Check content type
- `req.param(name)` - Get parameter by name
- `req.range(size[, options])` - Parse range header

**Response Methods (Express-Compatible):**

- `res.cookie(name, value[, options])` - Set cookie
- `res.clearCookie(name[, options])` - Clear cookie
- `res.redirect([status,] route)` - Redirect to route
- `res.render(template, model[, callback])` - Render template with model
- `res.json(json)` - Send JSON response
- `res.send(body)` - Send response
- `res.send(status)` - Send status code
- `res.status(code)` - Set status code
- `res.sendFile(path[, options][, fn])` - Send file
- `res.download(path[, filename][, options][, fn])` - Download file
- `res.links(links)` - Set Link header
- `res.location(path)` - Set Location header
- `res.type(type)` - Set Content-Type header
- `res.set(field[, value])` - Set response header
- `res.get(field)` - Get response header
- `res.append(field, value)` - Append header
- `res.vary(field)` - Add field to Vary header
- `res.attachment([filename])` - Set Content-Disposition
- `res.format(object)` - Content negotiation
- `res.jsonp(callback)` - Send JSONP response
- `res.end([data][, encoding])` - End response

**SPE-Specific Methods:**

- `app.render(template, model[, callback])` - Same as `res.render` on app
- `app.beforeEveryRender(fn)` - Execute before every render
- `app.afterEveryRender(fn)` - Execute after every render
- `app.postRenderCallbacks` - Object of post-render callbacks keyed by template name

### Backend API (Current - Express)

**Express Server Endpoints:**

- `GET /` - Render index template
- `GET /secondPage` - Render secondPage template
- `GET /pageWithForm` - Render form page
- `POST /pageWithForm` - Handle form submission
- `GET /pageWithDataRetrieval` - Render page with async data
- `POST /api/randomNumber` - JSON endpoint for random number
- Static file serving from `/public/` directory
- Template rendering via Teddy engine

**Express Configuration:**

```javascript
app.engine('html', require('teddy').__express)
app.set('views', 'mvc/views')
app.set('view engine', 'html')
app.use(express.static('public'))
app.use(bodyParser.urlencoded({ extended: true }))
```

### Backend API (Target - FastAPI)

**FastAPI Endpoints:**

- `GET /` - Render index template (Express-compatible handler)
- `GET /secondPage` - Render secondPage template
- `GET /pageWithForm` - Render form page
- `POST /pageWithForm` - Handle form submission
- `GET /pageWithDataRetrieval` - Render page with async data
- `GET /route/:withParam/:anotherParam` - Route with parameters
- `POST /api/randomNumber` - JSON endpoint for random number
- Static file serving from `/public/` directory
- Template rendering via Teddy (JavaScript engine) or Jinja2

**FastAPI Configuration:**

```python
from fastapi import FastAPI, Request, Response
from fastapi.staticfiles import StaticFiles
from fastapi.templating import Jinja2Templates
import uvicorn

app = FastAPI()
app.mount("/public", StaticFiles(directory="public"), name="public")

# Optional: Jinja2 integration
templates = Jinja2Templates(directory="mvc/views")
```

**Route Registration (Python):**

```python
# Method 1: Express-compatible decorator pattern
@app.get("/")
async def render_index(request: Request, response: Response):
    return await render_template("index", {})

# Method 2: YAML-driven route registration
@app.register_routes_from_yaml("mvc/routes.yaml")
```

**Teddy Integration (Python):**

```python
from teddy import TeddyEngine

# Option 1: PyExecJS
from PyExecJS import PyExecJS

engine = PyExecJS()
templates = load_templates("mvc/views")

def render_template(template_name, model):
    template = templates[template_name]
    js_code = f"""
        const teddy = require('teddy/client');
        teddy.setTemplate('{template_name}', `{template}`);
        module.exports = teddy.render('{template_name}', {model});
    """
    return engine.eval(js_code)

# Option 2: Subprocess
import subprocess
import json

def render_template(template_name, model):
    js_code = f"""
        const teddy = require('teddy/client');
        teddy.setTemplate('{template_name}', `{template}`);
        console.log(teddy.render('{template_name}', {model}));
    """
    result = subprocess.run(
        ['node', '-e', js_code],
        capture_output=True,
        text=True
    )
    return result.stdout
```

## Business Rules

### Route Matching Rules

1. **Case Sensitivity:** Routes are case-insensitive by default (`app.appVars['case sensitive routing'] = false`)
2. **Trailing Slash:** Trailing slashes are removed by default (`app.appVars['strict routing'] = false`)
3. **Wildcard Routes:** Express 4 uses `*`, Express 5 uses `*all` (configured via `app.expressVersion`)
4. **Query String Parsing:** Query strings are parsed into `req.query` object when `app.appVars['query parser'] = true`
5. **Subdomain Parsing:** Subdomains are parsed based on `app.appVars['subdomain offset']` (default: 2)

### Rendering Rules

1. **Default Target:** If no target specified, updates `body` element
2. **Targeted Updates:** `res.target` specifies CSS selector for element to update
3. **Multiple Targets:** `app.defaultTargets` array supports multiple update targets
4. **Append Mode:** `res.appendTargets` enables appending rather than replacing
5. **View Transitions:** Automatically uses View Transitions API when supported, falls back to CSS transitions
6. **Head Tag Updates:** New `<link>`, `<script>`, `<meta>`, `<style>` tags are added automatically
7. **External Resource Loading:** DOM update is delayed until external resources load to prevent FOUC

### Scroll and Focus Rules

1. **Scroll Memory:** Scroll position is saved per URL in `app.urls` object
2. **Scroll Restoration:** Previous scroll position is restored on page return
3. **Child Container Scroll:** Scroll positions are saved for elements with `id` attributes that scroll
4. **Scroll to Top:** Always scrolls to top on first visit or when `res.resetScroll` or `app.alwaysScrollTop` is set
5. **Focus Management:** Focus is set to `res.focus` element or first interactive element
6. **Accessibility:** Screen reader announcements via ARIA live region on page change
7. **Back/Forward Detection:** Classes `backButtonPressed` and `forwardButtonPressed` added to `<html>` element

### Form Handling Rules

1. **Form Hijacking:** All form submits with `action` attribute are intercepted
2. **Method Detection:** Form method is read from `method` attribute (default: GET)
3. **Body Parsing:** Form data is parsed into `req.body` object
4. **Submitter Data:** Button name/value is included in `req.body` if button has `name` attribute
5. **Empty Body:** Empty `req.body` is set to `undefined` in Express 5+ to match Express API

### Cookie Rules

1. **Cookie Reading:** Cookies are parsed from `document.cookie` into `req.cookies` object
2. **Cookie Writing:** `res.cookie()` writes to `document.cookie` with standard options
3. **Cookie Clearing:** `res.clearCookie()` sets expiration date to past to delete cookie
4. **Signed Cookies:** Signed cookies are not supported in browser context (stubbed with warning)

### Hook Execution Order

1. **Pre-render:** `app.beforeEveryRender()` → `res.beforeRender()`
2. **Delay:** Wait for `res.updateDelay` or `app.updateDelay` milliseconds
3. **Render:** Execute DOM update within view transition
4. **Post-render:** `res.afterRender()` → `app.afterEveryRender()`
5. **Post-render Callbacks:** Execute `app.postRenderCallbacks[template]()` → `app.postRenderCallbacks['*']()`
6. **Scroll:** Execute scroll restoration after animations complete

### Source File References

**Route Registration:** `single-page-express.js` lines 280-315
**Route Matching:** `single-page-express.js` lines 318-342
**Request Object:** `single-page-express.js` lines 141-180
**Response Object:** `single-page-express.js` lines 183-273
**Render Method:** `single-page-express.js` lines 485-736
**DOM Updates:** `single-page-express.js` lines 658-730
**Event Listeners:** `single-page-express.js` lines 741-780
**Express Server:** `sampleApps/express/server.js`
**Frontend Entry:** `sampleApps/express/browser.js`
**Route Definitions:** `sampleApps/express/mvc/routes.js`
**Express Complex Routes:** `sampleApps/express-complex/mvc/routes.js`

## UI Components

### Frontend Screens/Pages

**Sample App Pages:**

1. **Home Page** (`index.html`)
   - Purpose: Landing page with "hello world" content
   - Template: Teddy template with layout include
   - Route: `/`

2. **Second Page** (`secondPage.html`)
   - Purpose: Demonstrates variable passing to templates
   - Route: `/secondPage`
   - Model: `{variable: 'variable with contents: "hi there!"'}`

3. **Form Page** (`pageWithForm.html`)
   - Purpose: Demonstrates form submission handling
   - Route: `GET /pageWithForm`, `POST /pageWithForm`
   - Form data echoed back on POST

4. **Data Retrieval Page** (`pageWithDataRetrieval.html`)
   - Purpose: Demonstrates async model loading
   - Route: `/pageWithDataRetrieval`
   - Model: `{randomNumber: <async value>}`

5. **Parameter Page** (`params.html`)
   - Purpose: Demonstrates route parameter handling
   - Route: `/route/:withParam/:anotherParam`
   - Model: `{withParam: <param1>, anotherParam: <param2>}`

6. **Verbose Page** (`verbosePage.html`)
   - Purpose: Demonstrates verbose output
   - Route: `/verbosePage`

7. **Additional Head Tags Page** (`additionalHeadTags.html`)
   - Purpose: Demonstrates head tag management
   - Route: `/additionalHeadTags`

8. **Partial Page** (`partial.html`)
   - Purpose: Demonstrates partial template rendering
   - Route: `/routeUsingTarget`
   - Target: `body > article > *:first-child`

### UI Components

**Core Components:**

1. **Loading Indicator (Topbar)**
   - Purpose: Show progress during page transitions
   - Integration: Topbar library (https://buunguyen.github.io/topbar/)
   - Configuration: `app.topbarEnabled`, `app.topBarRoutes`, `app.topbarConfig`

2. **View Transition Container**
   - Purpose: Wrapper for animated DOM updates
   - Implementation: `document.startViewTransition()` API with CSS fallback
   - Configuration: `app.alwaysSkipViewTransition`, `res.skipViewTransition`

3. **ARIA Live Region**
   - Purpose: Announce page changes to screen readers
   - Implementation: Hidden `p` element with `aria-live="assertive"` and `aria-atomic="true"`
   - ID: `singlePageExpressDefaultRenderMethodAriaLiveRegion`

4. **Scroll Memory System**
   - Purpose: Remember scroll positions per page
   - Storage: `app.urls` object keyed by pathname
   - Includes: `scrollX`, `scrollY`, `scrollingChildContainers`

5. **Focus Manager**
   - Purpose: Set appropriate focus after page updates
   - Priority: `res.focus` → `[autofocus]` → first target element
   - Accessibility: Removes outline from non-interactive elements

### Template Components

**Layout Pattern:**

```html
<include src="layout">
  <arg pageContent>
    <p>Page content here</p>
  </arg>
</include>
```

**Teddy Template Features:**

- Include statements for layout inheritance
- Variable interpolation: `{variable}`
- Loops: `{on variable}` ... `{/on}`
- Conditionals: `{if condition}` ... `{/if}`
- Comments: `{! comment !}`

### Sample Application UI

**Express Sample UI Structure:**

- **CSS File:** `sampleApps/express/styles.css`
- **Layout Template:** `sampleApps/express/mvc/views/layout.html`
- **Page Templates:** Individual page templates extending layout
- **Browser Entry:** `sampleApps/express/browser.js` loads routes and initializes SPE
- **Server Entry:** `sampleApps/express/server.js` starts Express server and bundles frontend

## Authentication & Authorization

### Current State

**No Built-in Authentication:**

- The library provides routing, not a full authentication system
- Manual cookie management available via `res.cookie()` and `res.clearCookie()`
- `req.cookies` provides read access to cookies
- `req.signedCookies` is stubbed (not supported in browser context)

**Cookie Support:**

```javascript
// Set cookie
res.cookie('sessionToken', 'abc123', {
  maxAge: 3600000,  // 1 hour
  httpOnly: true,
  secure: true,
  sameSite: 'strict'
})

// Read cookie
const token = req.cookies.sessionToken

// Clear cookie
res.clearCookie('sessionToken')
```

**Cookie Options:**

- `domain` - Cookie domain
- `encode` - Encoding function (default: encodeURIComponent)
- `expires` - Expiration date
- `httpOnly` - HttpOnly flag
- `maxAge` - Max age in milliseconds
- `path` - Cookie path (default: "/")
- `partitioned` - Partitioned flag
- `priority` - Priority level
- `secure` - Secure flag
- `signed` - Signed flag (not supported, warns in development)
- `sameSite` - SameSite attribute

### Target State

**No Changes Required:**

- Authentication/authorization remains outside library scope
- Continue providing manual cookie management
- Maintain Express-compatible cookie API
- No built-in auth features will be added

**Future Integration Points:**

If authentication is added, it would be implemented as:

1. **User-Supplied Middleware:** Register middleware on routes
2. **Custom Integration:** Integrate with auth providers (JWT sessions, OAuth, etc.)
3. **Outside Scope:** Authentication is implementation detail, not library concern

**Example Middleware Pattern:**

```javascript
// Frontend middleware example
function authMiddleware(req, res, next) {
  if (!req.cookies.sessionToken) {
    res.redirect('/login')
  } else {
    next()
  }
}

app.get('/protected', authMiddleware, (req, res) => {
  res.render('protected', {})
})
```

**FastAPI Middleware Pattern:**

```python
# Backend middleware example
from fastapi import HTTPException, Request, Response

async def auth_middleware(request: Request, call_next):
    session_token = request.cookies.get("session_token")
    if not session_token:
        raise HTTPException(status_code=401, detail="Unauthorized")
    await call_next(request)

@app.middleware("http")
async def add_auth_middleware(request: Request, call_next):
    await auth_middleware(request, call_next)
```

## External Dependencies

### Frontend Dependencies (Preserved)

**Core Dependencies:**

- `path-to-regexp@8.3.0` - Route parser for Express 5+
  - Purpose: Parse and match route patterns (`/users/:id`)
  - Usage: `pathToRegexpMatch(route)` returns matching function

- `path-to-regexp-express4@0.1.12` - Route parser for Express 4 compatibility
  - Purpose: Support Express 4 route syntax
  - Usage: Returns regex for route matching

- `topbar@3.0.1` - Loading progress bar
  - Purpose: Show progress during page transitions
  - Usage: `app.topbar.show()` / `app.topbar.hide()`
  - Configuration: `app.topbarConfig({barColors: {...}})`

**Dev Dependencies:**

- `@playwright/test@1.58.2` - E2E browser testing
  - Purpose: Test routing, DOM updates, accessibility
  - Usage: `test()` / `expect()` from Playwright

- `standard@17.1.2` - JavaScript linter
  - Purpose: Enforce code style standards

- `webpack-cli@6.0.1` - Build tool
  - Purpose: Bundle library into multiple formats
  - Output: ESM, CommonJS, UMD, minified variants

**Optional Dependencies:**

- `teddy@1.1.4` - Templating engine (used in sample apps)
  - Purpose: Server-side and client-side templating
  - Compatibility: Express and browser contexts

### Backend Dependencies (Current - Express)

**Express Stack:**

- `express@5.2.1` - Web framework
  - Purpose: HTTP server and routing
  - Used in: `sampleApps/express/server.js`

- `body-parser` - Body parsing middleware
  - Purpose: Parse form submissions and JSON bodies
  - Usage: `app.use(bodyParser.urlencoded({ extended: true }))`

- `teddy@1.1.4` - Templating engine
  - Purpose: Server-side template rendering
  - Usage: `app.engine('html', teddy.__express)`

- `webpack` - Build tool for sample apps
  - Purpose: Bundle frontend JavaScript
  - Usage: `webpack(webpackConfig).run()`

### Backend Dependencies (Target - FastAPI)

**Required Dependencies:**

- `fastapi` (latest stable) - Web framework
  - Purpose: Modern async Python web framework
  - Features: Type hints, automatic OpenAPI docs, async/await

- `uvicorn[standard]` - ASGI server
  - Purpose: Run FastAPI applications
  - Features: Hot reload with `--reload` flag

- `pydantic` - Data validation
  - Purpose: Validate request/response data
  - Features: Type-based validation, automatic schemas

- `python-multipart` - Form data parsing
  - Purpose: Parse multipart form submissions
  - Usage: `FormData` handling in FastAPI

**Template Engine Dependencies:**

**Option 1: Keep Teddy**

- `PyExecJS` - JavaScript engine in Python
  - Purpose: Execute JavaScript template engine in Python
  - Alternative: Use subprocess to spawn Node.js process

**Option 2: Jinja2**

- `jinja2` - Python templating engine
  - Purpose: Server-side template rendering
  - Usage: `app.mount("/templates", Jinja2Templates(...))`

- `jinja-js` - Client-side Jinja2
  - Purpose: Jinja2 templates in browser
  - Note: May have subtle differences from Python Jinja2

**Build and Dev Dependencies:**

- `poetry` or `pip-tools` - Dependency management
  - Purpose: Python packaging and virtual environment

- `pytest` - Python testing framework
  - Purpose: Backend unit and integration tests

- `pytest-asyncio` - Async test support
  - Purpose: Test async FastAPI endpoints

- `black` or `ruff` - Code formatter/linter
  - Purpose: Python code style enforcement

### Testing Dependencies

**Frontend Tests (Preserved):**

- `@playwright/test@1.58.2` - E2E browser testing
  - Current test file: `test/tests.spec.js`
  - Coverage: Routing, DOM updates, scroll behavior, accessibility
  - Server: Launches Express complex sample for testing

**Backend Tests (New):**

- `pytest` - Python testing framework
  - Purpose: Test FastAPI endpoints and route registration
  - Location: `sampleApps/fastapi/tests/`

- `httpx` - Async HTTP client for testing
  - Purpose: Test FastAPI endpoints without server
  - Usage: `TestClient` from FastAPI

- `pytest-playwright` - Playwright integration for Python
  - Purpose: Run Playwright tests from Python test suite

**Integration Tests:**

- Combined frontend/backend testing
  - Verify isomorphic flow: server-rendered initial page, client-side navigation
  - Compare HTML output between Express and FastAPI implementations
  - Test route sharing between Python and JavaScript

### Documentation Dependencies

**Markdown Files:**

- `README.md` - Project overview and "Why this instead of..." comparisons
- `USAGE.md` - Usage documentation with examples
- `CONFIGURATION.md` - Configuration options reference
- `CONTRIBUTING.md` - Contribution guidelines
- `CHANGELOG.md` - Version history

**New Documentation Required:**

- FastAPI integration guide in USAGE.md
- Migration guide from Express to FastAPI backend
- Route and template sharing approach documentation
- Setup instructions for FastAPI development environment
- Docker configuration (new Dockerfile for FastAPI backend)

### Package Management

**npm (Frontend):**

- `package.json` - Frontend dependencies and scripts
- `npm publish` - Publish to npm registry
- Bundle formats: ESM, CommonJS, UMD, minified variants

**PyPI (Backend - New):**

- `pyproject.toml` - Python dependencies and metadata
- `poetry publish` or `twine upload` - Publish to PyPI
- Optional: Separate package for FastAPI integration

### Browser APIs

**Used by Library:**

- `window.history.pushState()` - History API for navigation
- `window.addEventListener('popstate')` - Back/forward button handling
- `document.startViewTransition()` - View Transitions API
- `window.scrollTo()` - Scroll position management
- `element.focus()` - Focus management
- `document.cookie` - Cookie access
- `document.querySelectorAll()` - DOM queries
- `DOMParser` - Parse HTML strings

**Supported Browsers:**

- Modern browsers with View Transitions API support
- Graceful degradation for browsers without View Transitions
- No increase in minimum browser requirements
