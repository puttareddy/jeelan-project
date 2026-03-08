# Data Model

## Entities

### App Instance
- **Fields:**
  - expressVersion: number (Target Express API version: 4 or 5+, default: 5)
  - appVars: object (Application settings for app.set()/app.get())
  - templatingEngine: object (Template engine instance with render() method)
  - templates: object (Loaded templates keyed by template name)
  - routes: object (Registered route patterns indexed by method#path)
  - routeCallbacks: object (Route handler functions indexed by method#path)
  - defaultTarget: string (CSS selector for default DOM update target)
  - defaultTargets: array (Array of CSS selectors for multiple update targets, default: ['body'])
  - beforeEveryRender: function (Hook to execute before every DOM update)
  - updateDelay: number (Milliseconds to delay after beforeRender before DOM update)
  - afterEveryRender: function (Hook to execute after every DOM update)
  - postRenderCallbacks: object (Per-template post-render callbacks keyed by template name or '*' for all)
  - topbarEnabled: boolean (Whether to use topbar loading indicator, default: true unless disabled)
  - topBarRoutes: array (Routes to apply topbar to, default: all if not specified)
  - topbar: object (Topbar instance if enabled)
  - alwaysSkipViewTransition: boolean (Never use View Transitions API)
  - alwaysScrollTop: boolean (Always scroll to top after render)
  - urls: object (Visited URLs with scroll position metadata)
  - locals: object (Local variables storage, stubbed)
  - mountpath: string (Mount path, stubbed as empty string)
- **Source:** `/Users/puttaiaharugunta/codebase/infi/single-page-express/single-page-express.js` (lines 5-39)

### Request Object (req)
- **Fields:**
  - app: object (Reference to app instance)
  - baseUrl: string (Base URL path, stubbed as empty string)
  - body: object or undefined (Form submission data or JSON body; undefined if empty in Express 5+)
  - cookies: object (Parsed cookies from document.cookie)
  - fresh: boolean (Cache freshness, stubbed as true)
  - hostname: string (Current hostname from window.location.hostname)
  - ip: string (Client IP, stubbed as '127.0.0.1')
  - ips: array (Client IPs, stubbed as empty array)
  - method: string (HTTP method: GET, POST, DELETE, PUT, etc.)
  - originalUrl: string (Original URL including query string)
  - params: object (Route parameters from path pattern, e.g., /users/:id → {id: '123'})
  - path: string (URL pathname from parsedUrl.pathname)
  - protocol: string (URL protocol: http: or https:)
  - query: object (Query string parameters parsed as key/value pairs)
  - res: object (Reference to response object)
  - route: object (Matched route information from path-to-regexp)
  - secure: boolean (True if protocol is https:)
  - signedCookies: object (Signed cookies, stubbed as empty object - not supported in browser)
  - stale: boolean (Cache staleness, stubbed as false)
  - subdomains: array (Parsed subdomains based on subdomain offset setting)
  - xhr: boolean (XHR indicator, stubbed as true)
  - singlePageExpress: boolean (True indicates SPE context)
  - backButtonPressed: boolean (True if back button was pressed)
  - forwardButtonPressed: boolean (True if forward button was pressed)
- **Methods:**
  - accepts(types): function (Check accepted content types, stubbed)
  - acceptsCharsets(charsets): function (Check accepted charsets, stubbed)
  - acceptsEncodings(encodings): function (Check accepted encodings, stubbed)
  - acceptsLanguages(langs): function (Check accepted languages, stubbed)
  - get(field): function (Get request header, stubbed)
  - is(type): function (Check content type, stubbed)
  - param(name): function (Get parameter by name, stubbed)
  - range(size[, options]): function (Parse range header, stubbed)
- **Source:** `/Users/puttaiaharugunta/codebase/infi/single-page-express/single-page-express.js` (lines 141-180, 377-451)

### Response Object (res)
- **Fields:**
  - app: object (Reference to app instance)
  - headersSent: boolean (Whether headers were sent, stubbed as false)
  - locals: object (Local variables storage, stubbed)
  - req: object (Reference to request object)
  - title: string or null (Override page title)
  - target: string or null (CSS selector for DOM update target)
  - appendTargets: boolean or null (Enable append mode instead of replace)
  - focus: string or null (CSS selector for focus element after render)
  - beforeRender: function or null (Hook to execute before DOM update)
  - afterRender: function or null (Hook to execute after DOM update)
  - removeMetaTags: boolean or null (Remove all meta tags from head)
  - removeStyleTags: boolean or null (Remove all style tags from head)
  - removeLinkTags: boolean or null (Remove all link tags from head)
  - removeScriptTags: boolean or null (Remove all script tags from head)
  - removeBaseTags: boolean or null (Remove all base tags from head)
  - removeTemplateTags: boolean or null (Remove all template tags from head)
  - removeHeadTags: boolean or null (Remove all non-title head tags)
  - skipViewTransition: boolean or null (Skip view transition animation)
  - updateDelay: number or null (Milliseconds to delay DOM update)
  - resetScroll: boolean or null (Reset scroll position for this request)
- **Methods:**
  - append(field, value): function (Append header, stubbed)
  - attachment([filename]): function (Set Content-Disposition, stubbed)
  - cookie(name, value[, options]): function (Set cookie in document.cookie)
  - clearCookie(name[, options]): function (Clear cookie by setting expiration to past date)
  - download(path[, filename][, options][, fn]): function (Download file, stubbed)
  - end([data][, encoding]): function (End response, stubbed)
  - format(object): function (Content negotiation, stubbed)
  - get(field): function (Get response header, stubbed)
  - json(json): function (Log JSON to console, stubbed - server only)
  - jsonp(callback): function (Send JSONP, stubbed)
  - links(links): function (Set Link header, stubbed)
  - location(path): function (Set Location header, stubbed)
  - redirect([status,] route): function (Redirect to route using handleRoute)
  - render(template, model[, callback]): function (Render template with model)
  - send(body): function (Send response, stubbed)
  - sendFile(path[, options][, fn]): function (Send file, stubbed)
  - sendStatus(code): function (Send status code, stubbed)
  - set(field[, value]): function (Set response header, stubbed)
  - status(code): function (Set status code, stubbed)
  - type(type): function (Set Content-Type header, stubbed)
  - vary(field): function (Add field to Vary header, stubbed)
- **Source:** `/Users/puttaiaharugunta/codebase/infi/single-page-express/single-page-express.js` (lines 183-273, 485-736)

### Route Definition
- **Fields:**
  - method: string (HTTP method: get, post, put, delete, all, etc.)
  - route: string (Route path pattern, e.g., '/', '/users/:id', '*all')
  - matcher: function or RegExp (Route matching function from path-to-regexp)
  - callback: function (Route handler function with (req, res) signature)
- **Storage Key:** `${method}#${route}` (e.g., 'get#/', 'post#/pageWithForm')
- **Source:** `/Users/puttaiaharugunta/codebase/infi/single-page-express/single-page-express.js` (lines 280-315)
- **Example Source:** `/Users/puttaiaharugunta/codebase/infi/single-page-express/sampleApps/express/mvc/routes.js`

### URL History Entry
- **Fields:**
  - scrollX: number (Horizontal scroll position)
  - scrollY: number (Vertical scroll position)
  - scrollingChildContainers: object (Scroll positions of child containers with id attributes)
    - *containerId*: { scrollX: number, scrollY: number } (Scroll position for each scrollable container)
- **Storage Key:** pathname (URL pathname from window.location.pathname)
- **Source:** `/Users/puttaiaharugunta/codebase/infi/single-page-express/single-page-express.js` (lines 352-366, 459-479)

### Model Data
- **Fields:** (Varies by model - arbitrary key/value pairs)
  - randomNumber: number (Example: random integer 1-10)
  - variable: string (Example: 'variable with contents: "hi there!"')
  - withParam: string (Example: route parameter value)
  - anotherParam: string (Example: route parameter value)
- **Pattern:** Async function returning data object
  - JavaScript: `async function () { return { ...data } }`
  - Python: `async def get_data(): return { ...data }`
- **Source:** `/Users/puttaiaharugunta/codebase/infi/single-page-express/sampleApps/express/mvc/models/server/getRandomNumber.js`

### Template Bundle
- **Fields:**
  - *templateName*: string (Teddy template source code as string)
- **Format:** JavaScript module export: `module.exports = { templateName: 'source code', ... }`
- **Generated At:** Server startup in Express samples
- **Source:** `/Users/puttaiaharugunta/codebase/infi/single-page-express/sampleApps/express/server.js` (lines 26-33)

### App Settings (appVars)
- **Fields:**
  - 'case sensitive routing': boolean (Enable case-sensitive routing, default: false)
  - env: string (Environment setting, default: 'production')
  - 'query parser': boolean (Enable query string parsing, default: true)
  - 'strict routing': boolean (Enable strict trailing slash handling, default: false)
  - 'subdomain offset': number (Subdomain parsing offset, default: 2)
- **Source:** `/Users/puttaiaharugunta/codebase/infi/single-page-express/single-page-express.js` (lines 73-77)

### Cookie Options
- **Fields:**
  - domain: string (Cookie domain attribute)
  - encode: function (Encoding function, default: encodeURIComponent)
  - expires: Date (Expiration date)
  - httpOnly: boolean (HttpOnly flag)
  - maxAge: number (Max age in milliseconds)
  - path: string (Cookie path, default: '/')
  - partitioned: boolean (Partitioned flag)
  - priority: string (Priority level: 'low', 'medium', 'high')
  - secure: boolean (Secure flag)
  - signed: boolean (Signed flag - not supported, warns in development)
  - sameSite: string (SameSite attribute: 'strict', 'lax', 'none')
- **Source:** `/Users/puttaiaharugunta/codebase/infi/single-page-express/single-page-express.js` (lines 196-223)

### Topbar Configuration
- **Fields:**
  - barColors: object (Color configuration for loading bar)
    - 0: string (Start color, default: 'rgba(0, 0, 0, .7)')
    - '1.0': string (End color, default: 'rgba(0, 0, 0, .7)')
- **Source:** `/Users/puttaiaharugunta/codebase/infi/single-page-express/single-page-express.js` (lines 27-35)

## Relationships Diagram

```
┌─────────────────┐
│   App Instance  │
│                 │
│  - routes[{}]  │
│  - routeCallbacks│
│  - urls[{}]    │
│  - appVars     │
│  - templates   │
└────────┬────────┘
         │
         │ has many
         ├──────────────────────────────────┐
         │                                  │
         │ 1:N                              │
    ┌────▼────────┐              ┌─────────▼────────┐
    │Route Definition│              │ URL History Entry  │
    │              │              │                 │
    │ - method     │              │ - scrollX        │
    │ - route      │              │ - scrollY        │
    │ - matcher    │              │ - childContainers │
    │ - callback   │              │                 │
    └────┬─────────┘              └───────────────────┘
         │
         │ creates per request
         ├────────────────┐
         │                │
         │ 1:1            │
    ┌────▼──────┐  ┌────▼──────┐
    │   Request  │  │  Response  │
    │            │  │            │
    │ - app*     │  │ - app*     │
    │ - body      │  │ - title    │
    │ - cookies   │  │ - target   │
    │ - params    │  │ - focus    │
    │ - query     │  │ - before... │
    │ - res*     │  │ - after...  │
    └─────┬──────┘  └─────┬──────┘
          │                │
          │ references      │ references
          └────────────────┘
         │
         │ uses
         ├─────────────────────┐
         │                   │
         │ 1:N               │ 1:1
    ┌────▼────────┐   ┌────▼─────────┐
    │ Model Data  │   │ Template Bundle│
    │             │   │              │
    │ {key: value}│   │ {name: code} │
    └─────────────┘   └──────────────┘

Legend:
  ──── Composition (has-a)
  ───→ Reference (points-to)
  ───| N:M Cardinality
  *Bidirectional reference
```

## Migration Notes

### No Persistent Data Models
- This is a client-side routing library with optional server integration
- No database migration required
- Data is ephemeral and passed through request/response cycles
- All state is maintained in memory (app instance, document, window)

### Template Sharing Strategy
- **Current:** Teddy templates used on both Express server and client-side
- **Migration:** Preserve Teddy templates for isomorphism
- **Options:**
  1. Keep Teddy with Python integration (PyExecJS or subprocess)
  2. Jinja2 with client-side Jinja2 (may have subtle differences)
  3. Template compilation to intermediate format
- **Recommendation:** Start with Option 1 (Keep Teddy) for MVP

### Route Definition Migration
- **Current:** JavaScript route files (e.g., routes.js) with function pattern
- **Migration:** Introduce YAML configuration for Python/JavaScript sharing
- **Example YAML:**
  ```yaml
  routes:
    - method: GET
      path: /
      template: index
    - method: GET
      path: /secondPage
      template: secondPage
      model:
        variable: 'variable with contents: "hi there!"'
  ```
- **Fallback:** Keep JavaScript route support via Python bridge for backward compatibility

### Data Model Pattern Migration
- **Current JavaScript:**
  ```javascript
  module.exports = async function () {
    return { randomNumber: Math.floor(Math.random() * 10) + 1 }
  }
  ```
- **Target Python:**
  ```python
  async def get_random_number():
      return {"randomNumber": random.randint(1, 10)}
  ```
- **Constraints:**
  - Must return dictionary/object with same key names
  - Must be async function/awaitable
  - Must be importable by both server and client contexts

### Template Bundling Migration
- **Current:** Express server bundles templates into `public/templates.js` at startup
- **Migration:** FastAPI server must generate equivalent bundle
- **Format:** JavaScript module export with template names mapping to source strings
- **Storage Location:** `public/` directory for static file serving

### Express to FastAPI Translation Guide

| Express Pattern | FastAPI Equivalent |
|---------------|---------------------|
| `app.get(path, handler)` | `@app.get(path)` decorator |
| `app.post(path, handler)` | `@app.post(path)` decorator |
| `req.params.id` | `path_params.id` in FastAPI handler |
| `req.query.key` | `query_params.key` in FastAPI handler |
| `req.body` | `await request.form()` or `request.json()` |
| `req.cookies` | `request.cookies` |
| `res.render(template, model)` | `templates.TemplateResponse(template, context)` |
| `res.redirect(path)` | `RedirectResponse(path)` |
| `res.cookie(name, value, opts)` | `response.set_cookie(name, value, opts)` |
| `app.use(middleware)` | `@app.middleware("http")` decorator |
| `app.set(name, value)` | App state configuration |

### Constraints to Preserve
1. **Frontend API Surface:** No changes to req/res object properties or methods
2. **Bundle Format:** Keep multi-format Webpack build (ESM, CommonJS, UMD)
3. **Template Compatibility:** Templates must render identically on server and client
4. **Route Parameters:** Support Express patterns (:param) and wildcards (*all/*)
5. **Query String Parsing:** Maintain Express-compatible query object structure
6. **Cookie Behavior:** Preserve document.cookie integration
7. **Scroll Position Memory:** Maintain app.urls storage pattern
8. **View Transitions API:** Keep browser-native view transition support
9. **Accessibility:** Preserve ARIA live region announcements and focus management
10. **Form Handling:** Support FormData parsing and submitter button data

### Data Validation
- Express: No built-in validation (req.body can be any shape)
- FastAPI: Pydantic models recommended for validation (optional for compatibility)
- Migration Strategy: Keep permissive validation to match Express behavior
- Form Data: Use `python-multipart` for multipart/form-data compatibility

### Static File Serving
- **Express:** `app.use(express.static('public'))`
- **FastAPI:** `app.mount('/public', StaticFiles(directory='public'))`
- **Files to Serve:**
  - `dist/single-page-express.*` (Bundled library)
  - `bundle.js` (Sample app frontend)
  - `templates.js` (Bundled templates)
  - `styles.css` (Stylesheets)

### History Management
- **Current:** Browser History API via `window.history.pushState()`
- **Migration:** Keep same approach (no FastAPI changes needed)
- **popstate Event:** Continue using for back/forward button detection
- **History Stack:** Maintain client-side stack for navigation direction detection

### Testing Data Models
- **Unit Tests:** Verify Python models return identical structure to JavaScript
- **Integration Tests:** Compare rendered HTML between Express and FastAPI
- **Playwright Tests:** All existing tests must pass with FastAPI backend
- **Test Data:** Same sample data for both implementations

### Backward Compatibility
- Keep Express sample apps functional during transition
- Provide migration path for existing users
- Support both JavaScript and Python route definitions
- Maintain npm package structure and exports
